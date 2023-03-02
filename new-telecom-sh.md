新的华为光猫远程重启脚本

```
#! /bin/bash

userName="telecomadmin"
passwd="nE7jA%255m"
gwUrl="http://192.168.1.1:8080/login.cgi"
deviceManUrl="http://192.168.1.1:8080/html/ssmp/devmanage/e8cdevicemanagement.asp"
rebootUrl="http://192.168.1.1:8080/html/ssmp/devmanage/set.cgi?x=InternetGatewayDevice.X_HW_DEBUG.SMP.DM.ResetBoard&RequestFile=html/ssmp/devmanage/e8cdevicemanagement.asp"
cookieFile="/tmp/gw-cookie-file.txt"
logFile="/tmp/gw-reboot.log"

function log_info () {
	DATE=$(date "+%Y-%m-%d %H:%M:%S")
	USER=$(whoami)
	echo "${DATE} ${USER} <$0> $@" >> ${logFile}
}

rm -rf "${cookieFile}"

curl -i -X POST -sS --connect-timeout 5 --max-time 10 -d "username=${userName}&psd=${passwd}" "${gwUrl}" -o /dev/null -c "${cookieFile}" >/dev/null
token=$(curl --connect-timeout 5 --max-time 10 -sS "${deviceManUrl}" -b "${cookieFile}" | grep -F "hwonttoken" | cut -d '"' -f 10)


if [ -z $token ]; then
	log_info "[ERROR]token 获取失败，可能登录没有成功！"
	exit 0
fi
log_info "[INFO]成功提取token: ${token}"

httpCode=$(curl --write-out "%{http_code}" -b "${cookieFile}" --connect-timeout 5 --max-time 10 -sS "${rebootUrl}" -X POST --data-raw "x.X_HW_Token=${token}")

if [ $httpCode -ne 200 ]; then
	log_info "[ERROR]执行了重启接口，返回的http code: ${httpCode}"
else
	log_info "[INFO]执行了重启接口，似乎重启成功了"
fi
```
