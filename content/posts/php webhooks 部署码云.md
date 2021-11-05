+++
title = "PHP webhook 部署码云"
date = 2021-11-04T19:12:06+08:00
draft = false
author = 'vdong'
categories = ['技术'] 
tags = ['php','webhook']
+++​

### 版本管理
有一个仓库（可以自建，也可以用线上的），将开发者本地的代码每次修改管理起来，可以查看修改记录，回滚等，常用的管理工具有svn 、git等，主流 git。


简要流程：

1. 修改代码
1. 提交修改到仓库

当然反过来也可以： 从仓库下载代码到本地  svn： 叫检出（ svn checkout path）； git 叫 拉取（git pull）

### 服务器同步
本地修改的代码要上传服务器，才能生效

简要流程：

1. 修改代码
1. 上传服务器（ftp，sftp： 直接本地上传；或者版本仓库里 拉取）


### 自动部署
可以发现第一步是一样的，我们可以把服务器当作一个新的本地环境，我们可以使用计算机，帮我们简化操作流程。

简化后的流程：

1. 修改代码
1. 提交修改到代码仓库
1. 从仓库拉取代码到服务器，即完成了上传服务器


以git为例，我们看一些实际部署（手动）：

1. 本地计算机执行： git commit -am 修改了一些文件
1. 本地计算机执行： git push origin/master 提交远程版本仓库
1. 服务器执行： git pull   服务器拉取版本仓库文件



webhook 帮助我们自动实现这个拉取过程。


### webhook自动部署的原理
![image.png](/imgs/1569555266752-382fa00b-2f12-4b46-b344-7c97742885b5.png)


webhooks：其实就是类似于触发事件


A事件发生 ----触发----> B事件
push事件  ----触发----> post : www.a.com/pull.php 


pull.php 完成工作： 切到网站根目录，git pull 拉取仓库代码


核心代码： shell_exec("cd /home/www/www.a.com && git pull 2>&1");


### 具体实现
#### 生成并部署SSH key

1. 服务器上生成ssh key
```shell
ssh-keygen -t rsa -C "xxxxx@xxxxx.com"  

# Generating public/private rsa key pair...
# 三次回车即可生成 ssh key
```

2. 查看公钥并添加的码云
```shell
cat ~/.ssh/id_rsa.pub
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6eNtGpNGwstc....
```
复制以上公钥，添加到仓库 "部署公钥管理"。
![image.png](/imgs/1571716634885-6da18983-92e0-4a7d-b2aa-255889f0e068.png)

3. 测试出现如下表示添加成功

```shell
ssh -T git@gitee.com
Hi Anonymous! You've successfully authenticated, but GITEE.COM does not provide shell access.
Note: Perhaps the current use is DeployKey.
Note: DeployKey only supports pull/fetch operations
```

#### 服务器初始化clone仓库
```shell
$ cd /home/wwwroot/           # 代码存放目录
$ git clone git@gitee.com:vdong/t.whaoot.com.git
```

#### 准备php代码


```php
<?php
// 本地仓库路径
$local = '/home/wwwroot/t.whaoot.com';

// 安全验证字符串，为空则不验证
$token = '111111';


// 如果启用验证，并且验证失败，返回错误
$httpToken = isset($_SERVER['HTTP_X_GITEE_TOKEN']) ? $_SERVER['HTTP_X_GITEE_TOKEN'] : '';
if ($token && $httpToken != $token) {
    header('HTTP/1.1 403 Permission Denied');
    die('Permission denied.');
}

// 如果仓库目录不存在，返回错误
if (!is_dir($local)) {
    header('HTTP/1.1 500 Internal Server Error');
    die('Local directory is missing');
}

//如果请求体内容为空，返回错误
$payload = file_get_contents('php://input');
if (!$payload) {
    header('HTTP/1.1 400 Bad Request');
    die('HTTP HEADER or POST is missing.');
}

/*
 * 这里有几点需要注意：
 *
 * 1.确保PHP正常执行系统命令。写一个PHP文件，内容：
 * `<?php shell_exec('ls -la')`
 * 在通过浏览器访问这个文件，能够输出目录结构说明PHP可以运行系统命令。
 * 否则 修改web服务器上php.ini的 disable_functions 列表，去掉 shell_exec; 重启php-fpm服务
 *
 * 2、PHP一般使用www-data或者nginx用户运行，PHP通过脚本执行系统命令也是用这个用户，
 * 所以必须确保在该用户家目录（一般是/home/www-data或/home/nginx）下有.ssh目录和
 * 一些授权文件，以及git配置文件，如下：
 * ```
 * /root/.ssh
 * + .ssh
 *   - authorized_keys
 *   - config
 *   - id_rsa
 *   - id_rsa.pub
 *   - known_hosts
 * - .gitconfig
 * ```
 *
 * 3.在执行的命令后面加上2>&1可以输出详细信息，确定错误位置
 *
 * 4.git目录权限问题。比如：
 * `fatal: Unable to create '/data/www/html/awaimai/.git/index.lock': Permission denied`
 * 那就是PHP用户没有写权限，需要给目录授予权限:
 * ``
 * sudo chown -R www:www /data/www/html/awaimai`
 * sudo chmod -R g+w /data/www/html/awaimai
 * ```
 *
 * 5.SSH认证问题。如果是通过SSH认证，有可能提示错误：
 * `Could not create directory '/.ssh'.`
 * 或者
 * `Host key verification failed.`
 *  .ssh 要放到对应的用户目录下
 * 例如： php 是以 www 用户运行的， /home/www/.ssh
 *
 */

#以可写权限打开git_new.log文件，用于记录git日志
$fs = fopen('/home/www/git.t.whaoot.com.log', 'a');
fwrite($fs, '================ Update Start ==============='.PHP_EOL.PHP_EOL);
#获取请求端的IP
$client_ip = $_SERVER['REMOTE_ADDR'];
#将时间与请求端IP写入日志文件
fwrite($fs, 'Request on ['.date("Y-m-d H:i:s").'] from ['.$client_ip.']'.PHP_EOL);
#执行shell命令cd到网站根目录执行git pull操作并把返回信息赋值给变量output
$output=shell_exec("cd {$local} && git pull 2>&1");
#将日志信息写入日志文件中
fwrite($fs, 'Info:'. $output.PHP_EOL);
fwrite($fs,PHP_EOL. '================ Update End ==============='.PHP_EOL.PHP_EOL);
#关闭日志文件
echo $output
die("done " . date('Y-m-d H:i:s', time()));
$fs and fclose($fs);
```
#### 绑定webhooks
去码云设置 WebHooks，点击测试


#### 参考链接：
[https://www.awaimai.com/2203.html](https://www.awaimai.com/2203.html)

[https://blog.csdn.net/gbenson/article/details/84346696](https://blog.csdn.net/gbenson/article/details/84346696)
