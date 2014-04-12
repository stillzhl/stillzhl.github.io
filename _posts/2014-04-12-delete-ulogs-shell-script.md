---
layout: post
title: 维护ttserver ulog文件的脚本
---

目前公司的ttserver放在了一块ssd上，由于ssd的容量比较小，所以要定期删除用来主从备份的ulog文件。
下面是脚本:

     1 #!/bin/bash
     2
     3 remove_old_ulog()
     4 {
     5     now_stamp=`date '+%s'`
     6     one_week_secs=604800
     7     one_week_ago=`expr $now_stamp - $one_week_secs`
     8
     9     for ulog_file in `ls $1`
    10         do
    11             modified_date_stamp=`stat -c "%Y" $ulog_file | awk '{print $1}'`
    12             modified_date=`stat -c "%y" $ulog_file | awk '{print $1}'`
    13             if [ $modified_date_stamp -lt $five_days_ago ]
    14             then
    15                 rm -f $ulog_file
    16                 echo "$ulog_file $modified_date deleted"
    17             fi
    18         done
    19 }
    20
    21 ulog_dir="/path/to/ttserver/ulog"
    22
    23 echo "Deleting old ulogs\n"
    24 cd $ulog_dir
    25 remove_old_ulog $ulog_dir
