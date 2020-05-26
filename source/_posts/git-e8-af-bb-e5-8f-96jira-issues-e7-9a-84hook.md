---
title: Git读取Jira issues的hook
url: 390.html
id: 390
categories:
  - Git
  - Linux
date: 2018-03-02 10:43:21
tags:
---

为了方便填写git commit message，写了一个脚本用于读取jira上的issues，并列出选择作为commit message。 用到了git hook中的prepare-commit-msg 下面提供bash shell和python3两个版本 1.bash shell 此版本需要安装jq。

    #!/bin/bash
    USERNAME=`git config --get user.name`
    if [ "$2" = "message" -o "$2" = "commit" ]
    then
        exit 0
    fi
    exec 2> /dev/null
    ISSUS=`curl -u jira:jira "http://localhost:8081/rest/api/2/search?jql=status+in+(Reopened,%20%22To%20Do%22,%20%22In%20Progress%22)%20AND%20assignee%20in%20($USERNAME)%20ORDER%20BY%20updated%20DESC" | jq '.issues[]|{summary: .fields.summary, key: .key}'`
    KEYS=(`echo $ISSUS | jq '.key'`)
    SUMMARYS_TMP=("`echo $ISSUS | jq '.summary'`")
    IFS_old=$IFS
    IFS=$'\n'
    SUMMARYS=($SUMMARYS_TMP)
    IFS='"'
    number=${#KEYS[@]}
    declare -a ISSUSES='()'
    for((i=0;i<$number;i++))
    do
        ISSUSES[$i]="${KEYS[$i]}${SUMMARYS[$i]}"
    done
    for((j=0;j<${#ISSUSES[@]};j++))
    do
        let k=$j+1
        echo $k.${ISSUSES[$j]}
    done
    exec < /dev/tty
    exec 2>&1
    read -p "请输入编号："
    let answer=$REPLY-1
    echo ${ISSUSES[$answer]} > $1
    IFS=$IFS_old
    

2.python3

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-
    
    import sys, os, re
    from subprocess import check_output
    import requests
    import json
    
    def query_report(user):
        s = requests.Session()
        s.post('http://localhost:8081/rest/auth/1/session', json={"username":"jira","password":"jira"})
        url = 'http://localhost:8081/rest/api/2/search?jql=status+in+(Reopened,%20%22To%20Do%22,%20%22In%20Progress%22)%20AND%20assignee%20in%20(' + user +')%20ORDER%20BY%20updated%20DESC'
        reports = s.get(url)
        return reports.json()
    
    sys.stdin = open('/dev/tty')
    # 收集参数
    commit_msg_filepath = sys.argv[1]
    if len(sys.argv) > 2:
        commit_type = sys.argv[2]
    else:
        commit_type = ''
    if len(sys.argv) > 3:
        commit_hash = sys.argv[3]
    else:
        commit_hash = ''
    
    if commit_type == "message":
        sys.exit(0)
    # 检测我们所在的分支
    branch = check_output(['git', 'symbolic-ref', '--short', 'HEAD']).strip()
    branch = str(branch, 'utf-8')
    print("On branch '%s'" % branch)
    username = check_output(['git', 'config', '--get', 'user.name']).strip()
    username = str(username, 'utf-8')
    messages = []
    if branch.startswith('master'):
        j = query_report(username)
        if j["total"] > 0:
            issues = j["issues"]
            for i in range(len(issues)):
                print(str(i+1) + ':' + issues[i]["key"] + " " + issues[i]["fields"]["summary"])
                messages.append(issues[i]["key"] + " " + issues[i]["fields"]["summary"]);
        number = input("请输入编号: ")
        message = messages[int(number) - 1]
        with open(commit_msg_filepath, 'w') as f:
            f.seek(0, 0)
            f.write(message)