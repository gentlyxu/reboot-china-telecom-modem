# reboot-china-telecom-modem
重启电信光猫的shell脚本

没有找到一个可以直接用的脚本，大晚上的撸个轮子吧。。。


```
#! /bin/bash

userName="useradmin"
passwd="xxxx"
gwUrl="http://192.168.1.1/cgi-bin/luci/"
cookieFile="/tmp/gw-cookie-file.txt"
logFile="/tmp/gw-reboot.log"

function log_info () {
	DATE=$(date "+%Y-%m-%d %H:%M:%S")
	USER=$(whoami)
	echo "${DATE} ${USER} <$0> $@" >> ${logFile}
}

rm -rf "${cookieFile}"
curl -X POST -sS --connect-timeout 5 --max-time 10 -d "username=${userName}&psd=${passwd}" "${gwUrl}" -o /dev/null -c "${cookieFile}" >/dev/null
token=$(curl --connect-timeout 5 --max-time 10 -sS "${gwUrl}" -b "${cookieFile}" | grep -F "/admin/logout', data: { token: '" | cut -d "'" -f 6)


if [ -z $token ]; then
	log_info "[ERROR]token 获取失败，可能登录没有成功！"
	exit 0
fi
log_info "[INFO]成功提取token: ${token}"

httpCode=$(curl --write-out "%{http_code}" -b "${cookieFile}" --connect-timeout 5 --max-time 10 -sS "${gwUrl}/admin/reboot" -X POST --data-raw "token=${token}")

if [ $httpCode -ne 200 ]; then
	log_info "[ERROR]执行了重启接口，返回的http code: ${httpCode}"
else
	log_info "[INFO]执行了重启接口，似乎重启成功了"
fi

```

把这个脚本丢到crontab里就行了

啊，适用于这种光猫

![电信光猫](https://raw.githubusercontent.com/gentlyxu/reboot-china-telecom-modem/master/t.jpg)




