+++
title = "Shell编程-2"
date = 2023-09-25T10:23:40+08:00
draft = false
author = 'vdong'
categories = [ '技术']
tags = ['unix','shell']

+++

## Shell解析yml

代码实现如下：

```shell
# 解析 yml , 数组自动添加序号
# 
### 例如：
### yml 如下：
### global:
###   input:
###     - "main.c"
###     - "main.h"
###   flags: [ "-O3", "-fpic" ]
###   sample_input:
###     -  { property1: value, property2: "value2" }
###     -  { property1: "value3", property2: 'value 4' }
### 
### 执行： eval $(parse_yaml "config.yml"）
###
### 解析结果如下，相当于生成了如下代码:
### global_input_1="main.c"
### global_input_2="main.h"
### global_flags_1="-O3"
### global_flags_2="-fpic"
### global_sample_input_1_property1="value"
### global_sample_input_1_property2="value2"
### global_sample_input_2_property1="value3"
### global_sample_input_2_property2="value 4"
### 
### 最后可以使用间接变量引用，访问变量
###
function parse_yaml {
   local prefix=$2
   local s='[[:space:]]*' w='[a-zA-Z0-9_]*' fs=$(echo -e '\034')
   sed -ne "s|,$s\]$s\$|]|" \
        -e ":1;s|^\($s\)\($w\)$s:$s\[$s\(.*\)$s,$s\(.*\)$s\]|\1\2: [\3]\n\1  - \4|;t1" \
        -e "s|^\($s\)\($w\)$s:$s\[$s\(.*\)$s\]|\1\2:\n\1  - \3|;p" $1 | \
   sed -ne "s|,$s}$s\$|}|" \
        -e ":1;s|^\($s\)-$s{$s\(.*\)$s,$s\($w\)$s:$s\(.*\)$s}|\1- {\2}\n\1  \3: \4|;t1" \
        -e    "s|^\($s\)-$s{$s\(.*\)$s}|\1-\n\1  \2|;p" | \
   sed -ne "s|^\($s\):|\1|" \
        -e "s|^\($s\)-$s[\"']\(.*\)[\"']$s\$|\1$fs$fs\2|p" \
        -e "s|^\($s\)-$s\(.*\)$s\$|\1$fs$fs\2|p" \
        -e "s|^\($s\)\($w\)$s:$s[\"']\(.*\)[\"']$s\$|\1$fs\2$fs\3|p" \
        -e "s|^\($s\)\($w\)$s:$s\(.*\)$s\$|\1$fs\2$fs\3|p" | \
   awk -F$fs '{
      indent = length($1)/2;
      vname[indent] = $2;
      for (i in vname) {if (i > indent) {delete vname[i]; idx[i]=0}}
      if(length($2)== 0){  vname[indent]= ++idx[indent] };
      if (length($3) > 0) {
         vn=""; for (i=0; i<indent; i++) { vn=(vn)(vname[i])("_")}
		 gsub(/\s*#.*$/, "", $3);
         printf("%s%s%s=\"%s\"\n", "'$prefix'",vn, vname[indent], $3);
      }
   }'
}
```

用法：

`yml` 文件内容如下：

```yml
# yml
elasticsearch: 
  - setup/elastic/ilm-policies/my-ilm-policy.request
  - setup/elastic/index-templates/switch.request
mysql:
  host: 127.0.0.1
```

解析代码如下：

```shell
elastic_set_file="${BASH_SOURCE[0]%/*}"/elastic/setup-elasticsearch.yml
if [[ ! -f "${elastic_set_file:-}" ]]; then
	sublog "No setup-elastic.yml file found, skipping"
	continue
fi
# 解析 yml 并添加前缀，避免变量名已存在
eval $(parse_yaml "$elastic_set_file" "config_")
# 会生成如下变量
config_elasticsearch_1="setup/elastic/ilm-policies/my-ilm-policy.request"
config_elasticsearch_2="setup/elastic/index-templates/switch.request"
config_mysql_host="127.0.0.1"
```

## bash 动态变量名（间接变量引用）

```shell
# 假设有两个变量
var1="Hello"
index=1

# 使用间接变量引用来构造动态变量名
dynamic_var="var${index}"

# 使用动态变量名访问变量的值
echo "${!dynamic_var}"  # 输出：Hello

# 例如：有多个递增变量
# global_input_1="main.c"
# global_input_2="main.h"
# ...
# global_input_100="xxx.c"
for idx in {1..100}; do
	variable_name="global_input_$idx"
	if [ ! -v "$variable_name" ]; then
		break
	if
	
	file_name="${!variable_name}"
	# more code
done
```

## bash 参数扩展

参数扩展是 Bash 中强大的字符串操作工具，可以用于各种字符串处理任务。

以下是一些常用的参数扩展写法示例：

1. `${varname#substring}`：删除最短匹配前缀。
2. `${varname##substring}`：删除最长匹配前缀。
3. `${varname%suffix}`：删除最短匹配后缀。
4. `${varname%%substring}`：删除最长匹配后缀。
5. `${varname/old/new}`：将第一个匹配的 "old" 子字符串替换为 "new"。
6. `${varname//old/new}`：将所有匹配的 "old" 子字符串替换为 "new"。
7. `${#varname}`：获取字符串的长度。
8. `${varname[offset,length]}`：从字符串中提取子字符串，从 `offset` 位置开始，长度为 `length`。

## bash 迭代关联数组或关联列表

```shell
declare -A users_passwords
users_passwords=(
	[logstash_internal]="${LOGSTASH_INTERNAL_PASSWORD:-}"
	[kibana_system]="${KIBANA_SYSTEM_PASSWORD:-}"
	[metricbeat_internal]="${METRICBEAT_INTERNAL_PASSWORD:-}"
	[filebeat_internal]="${FILEBEAT_INTERNAL_PASSWORD:-}"
	[heartbeat_internal]="${HEARTBEAT_INTERNAL_PASSWORD:-}"
	[monitoring_internal]="${MONITORING_INTERNAL_PASSWORD:-}"
	[beats_system]="${BEATS_SYSTEM_PASSWORD=:-}"
)

for user in "${!users_passwords[@]}"; do
	echo "User '$user'"
	if [[ -z "${users_passwords[$user]:-}" ]]; then
		echo 'No password defined, skipping'
		continue
	fi
	
	# code more ...
done
```

## 数组展开

```shell
args=("http://example.com" "-H" "User-Agent: MyUserAgent" "--data" "param1=value1" "--header" "Content-Type: application/json")
curl "${args[@]}"

# 与下边形式等价，避免了拼接字符串
curl http://example.com -H "User-Agent: MyUserAgent" --data "param1=value1" --header "Content-Type: application/json"
```

