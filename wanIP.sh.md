```bash
#! /bin/bash

userName="telecomadmin"
passwd="nE7jA%255m"
gwUrl="http://192.168.1.1:8080/login.cgi"
wanIpInfoUrl="http://192.168.1.1:8080/html/bbsp/common/wan_list_e8c.asp?20220517220910422869193943"
cookieFile="/tmp/gw-cookie-file.txt"
logFile="/tmp/new-ip-change.log"

function log_info () {
        DATE=$(date "+%Y-%m-%d %H:%M:%S")
        USER=$(whoami)
        echo "${DATE} ${USER} <$0> $@" >> ${logFile}
}

rm -rf "${cookieFile}"

curl -i -X POST -sS --connect-timeout 5 --max-time 10 -d "username=${userName}&psd=${passwd}" "${gwUrl}" -o /dev/null -c "${cookieFile}" >/dev/null
wanIP=$(curl --connect-timeout 5 --max-time 10 -sS "${wanIpInfoUrl}" -b "${cookieFile}" | awk '/^var PPPWanList = new/{split($0, item, "\",\"");gsub("\\\\x2e",".", item[68]);print item[68]}')


if [ -z $wanIP ]; then
        log_info "[ERROR]wanIP 获取失败，可能登录没有成功！"
        exit 0
fi
log_info "[INFO]成功获取外网IP: ${wanIP}"
echo $wanIP
```
