---
title: 从AWS S3下载指定日期范围的日志到本地
date: 2018-02-18 20:48:41
categories: Shell脚本
tags: Shell
---

**本脚本主要包括如下要点：**

  * Shell脚本日期循环;
  * AWS S3命令行使用;
  * 通过正则进行日期合法性校验;
  * Shell命令执行无限重试，直到成功;
```bash
# !/bin/bash
# @Time    : 2017/10/11 下午3:20
# @Author  : qianghaohao
# @Mail    : codenutter@foxmail.com
# @File    : s3log_downloader.sh

#---------------- 脚本参数说明 --------------
# $1: 业务类型
# $2: 日志类型
# $3: 开始时间: 年-月-日 ex:2017-10-10
# $4: 结束时间: 年-月-日 ex:2017-10-10
#--------------------------------------------
PROFILE=s3_key_profile
#  下载失败则无限重试
function repeat()
{
    while true
    do
        $@ && return
        sleep 5
    done
}

# -----------------------------------------------------
#  合并一个目录下的文件到一个文件,
#  函数参数说明:
#  $1:日志文件夹
#  $2:合并后文件名:%{business}_%{type}-%{date}.log
# -----------------------------------------------------
function merge_file()
{
    cd $1 && ls -rt | xargs cat > $2 && rm -rf $1
}

datebeg="$3"
dateend="$4"
#  检查日期格式是否正确
if [ -z "`echo "$datebeg" | grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}$"`" -o -z "`echo "$dateend" | grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}$"`" ]; then
    echo "Error: 日期格式不合法，请输入正确的日期格式[ex: 2017-09-09]..."
    exit -1
fi

beg_s=`date -d "$datebeg" +%s`
end_s=`date -d "$dateend" +%s`

while [ "$beg_s" -le "$end_s" ]
do
     day=`date -d @$beg_s +"%Y-%m-%d"`
     s3_log_path="s3://s3-path/$1/$2/$day"
     local_log_path="/home/path/$1/$2/$day"
     local_log_file="/home/path/$1/$2/${1}-${2}-${day}.log"
     #   后台执行任务(非阻塞) 类似于多进程并发下载
     repeat aws s3 cp --recursive ${s3_log_path} ${local_log_path} --profile=$PROFILE && merge_file ${local_log_path} ${local_log_file} &
     beg_s=$((beg_s+86400))
done
wait
echo "=============================> 下载完成 ============================"
```
