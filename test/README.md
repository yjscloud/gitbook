---
description: test
---

# test

```text
#!/bin/bash

# 定义颜色
RESTORE=$(echo -en '\033[0m')
RED=$(echo -en '\033[00;31m')
GREEN=$(echo -en '\033[00;32m')
YELLOW=$(echo -en '\033[00;33m')
CYAN=$(echo -en '\033[00;36m')
# 判定日志目录
BASE_DIR=/var/log/everyday
TEMP_DIR=/tmp/directory.txt
DATE=`date +%F`
ls -lth ${BASE_DIR} > ${TEMP_DIR}

if [ `grep -c ${DATE} ${TEMP_DIR}` -eq 1 ];then
        DIR=${BASE_DIR}/${DATE}
else
        DATE1=`grep ${DATE} ${TEMP_DIR}|awk '{print $9}'|sed -n 1p`
        DIR=${BASE_DIR}/${DATE1}
fi

#==============================================
echo
echo "1. check ping of the physical server."
DIR1=${DIR}/ping-result.txt
# 自定义ping巡检节点数量
PING_NUM=86
# txt文件显示节点数量
CHECK_NUM=`grep -Ec '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR1`
# 成功执行节点数量
SUCCESS_NUM=`grep -Ec 'SUCCESS' $DIR1`

if [ ${SUCCESS_NUM} -eq ${PING_NUM} ];then
	echo ${CYAN}ALL_MACHINE_NUM: ${PING_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE}  ${GREEN}SUCCESS: ${SUCCESS_NUM}${RESTORE} '>>>>>' ${GREEN}PING ALL MACHINE IS OK.${RESTORE}
else
	# ping失败IP及数量
	FAILED_IP=`grep -E 'UNREACHABLE|FAILED' $DIR1|awk '{print $1}'`
	FAILED_NUM=`grep -Ec 'UNREACHABLE|FAILED' $DIR1`
	echo ${CYAN}ALL_MACHINE_NUM: ${PING_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE}  ${GREEN}SUCCESS: ${SUCCESS_NUM}${RESTORE}  ${RED}FAILED: ${FAILED_NUM}${RESTORE} IP: ${RED}${FAILED_IP}${RESTORE}
fi

#==============================================
echo
echo "2. check server network cards."
DIR2_1=${DIR}/wangka-region2.txt
DIR2_2=${DIR}/wangka-other-region.txt
# 自定义巡检节点数量
NETWORK_NOTE_NUM=81
# txt文件显示节点数量
CHECK_NUM1=`grep -Ec '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR2_1`
CHECK_NUM2=`grep -Ec '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR2_2`
let CHECK_NUM=CHECK_NUM1+CHECK_NUM2
# 获取txt文件中的IP存入数组
ALL_IP1=(`grep -E '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR2_1 |awk '{print $1}'`)
ALL_IP2=(`grep -E '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR2_2 |awk '{print $1}'`)
# 参数初始化
i=0
j=0
SUCCESS=()
FAILED=()
# region2网卡全部用完，DOWN的数量为0
for IP in ${ALL_IP1[@]}
do
	if [  `grep ${IP} ${DIR2_1}|grep -Ec 'FAILED'` -ne 1 ];then
		FAILED[$i]=${IP}
		let i=i+1
	else
		NUM=`grep -A1 ${IP} ${DIR2_1}|sed -n 2p|tr -d '^M|\r'`
		if [ ${NUM} -eq 0 ];then
			SUCCESS[$j]=${IP}
			let j=j+1
		else
			FAILED[$i]=${IP}
                	let i=i+1
		fi
	fi
done

# 其他节点剩余2张网卡，DOWN的数量为2
for IP in ${ALL_IP2[@]}
do
	if [  `grep ${IP} ${DIR2_2}|grep -Ec 'SUCCESS'` -ne 1 ];then
		FAILED[$i]=${IP}
		let i=i+1
	else
		NUM=`grep -A1 ${IP} ${DIR2_2}|sed -n 2p|tr -d '^M|\r'`
		if [ ${NUM} -eq 2 ];then
			SUCCESS[$j]=${IP}
			let j=j+1
		else
			FAILED[$i]=${IP}
                	let i=i+1
		fi
	fi
done

if [ ${#FAILED[*]} -eq 0 ];then
	echo ${CYAN}NETWORK_NOTE_NUM: ${NETWORK_NOTE_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SUCCESS: ${#SUCCESS[*]}${RESTORE} '>>>>>' ${GREEN}NETWORK_CARD CHECK IS OK.${RESTORE}
else
	echo ${CYAN}NETWORK_NOTE_NUM: ${NETWORK_NOTE_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SUCCESS: ${#SUCCESS[*]}${RESTORE} ${RED}FAILED: ${#FAILED[*]}${RESTORE} IP: ${RED}${FAILED[@]}${RESTORE}
fi

#==============================================
echo
echo "3. check region VIP only in one node."
DIR3=${DIR}/vip-result.txt
# 巡检节点数量
SERVER_NUM=14
# 成功巡检数量
CHECK_NUM=`grep -c 'SUCCESS' ${DIR3}`

j=1
for i in 10.123.1.11 10.123.1.12 10.123.1.13 10.123.1.14 10.70.249.11
do
    if [ `grep -c $i ${DIR3}` -ne 1 ];then
        ERR_IP[$j]=$i
    fi
    let j++
done

if [ ${#ERR_IP[*]} -eq 0 ];then
	echo ${CYAN}SERVER_NUM: ${SERVER_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SUCCESS: ${CHECK_NUM}${RESTORE} '>>>>>' ${GREEN}VIP CHECK IS OK.${RESTORE}
else
	let SUCCESS_NUM=${CHECK_NUM}-${#ERR_IP[*]}
	echo ${CYAN}SERVER_NUM: ${SERVER_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SUCCESS: ${SUCCESS_NUM}${RESTORE} ${RED}FAILED: ${#ERR_IP[*]} ${RESTORE}IP: ${RED}${ERR_IP[@]}${RESTORE}
fi
#===============================================
echo
echo "4. check galera cluster."
DIR4=${DIR}/galera-result.txt
# 自定义garela巡检节点数量
GALERA_CLUSTER_NUM=12
# 成功执行节点数量
SUCCESS_NUM=`grep -Ec 'HTTP/1.1 200 OK' $DIR4`
# txt文件显示节点数量
CHECK_NUM=`grep -Ec '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR4`

if [ ${SUCCESS_NUM} -eq ${GALERA_CLUSTER_NUM} ];then
	echo ${CYAN}GALERA_CLUSTER_NUM: ${GALERA_CLUSTER_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SCCESS: ${SUCCESS_NUM}${RESTORE} '>>>>>' ${GREEN}GALERA CLUSTER CHECK IS OK.${RESTORE}
else
	# 获取txt文件中全部IP存入数组
	ALL_IP=(`grep -E '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR4|awk '{print $1}'`)
	i=0
	for IP in ${ALL_IP[@]}
	do
		TEXT_OK=`grep -A1 $IP $DIR4|sed -n 2p|tr -d '^M|\r'`
		if [ "${TEXT_OK}" == 'HTTP/1.1 200 OK' ];then
			:
		else
			ERR_IP[$i]=$IP
			let i=i+1
		fi
	done
	echo ${CYAN}GALERA_CLUSTER_NUM: ${GALERA_CLUSTER_NUM}${RESTORE} ${YELLOW}CHECK_NUM: ${CHECK_NUM}${RESTORE} ${GREEN}SCCESS: ${SUCCESS_NUM}${RESTORE} ${RED}FAILED: ${i}${RESTORE} IP: ${RED}${ERR_IP[@]}${RESTORE}
fi

#================================================
echo
echo "5.1 check rabbitmq cluster."
for i in {1..5}
do
	DIR5_1=${DIR}/rabbitmq-cluster-region$i.txt
	# txt文件中执行成功数
	SUCCESS_NUM=`grep -Ec 'SUCCESS' ${DIR5_1}`
	# 获取执行节点IP
	NODE_IP=`cat ${DIR5_1}|tr -d '\n'|awk '{print $1}'`
	if [ ${SUCCESS_NUM} -eq 1 ];then
		# 获取运行的集群节点
		RUNNING_NODES=`cat ${DIR5_1}|tr -d '\n'|awk -F'{|}' '{print $6}'|awk -F',' '{print $2"," $3"," $4}'`
		# 获取集群分区信息
        	PARTITION=`cat ${DIR5_1}|tr -d '\n'|awk -F'{|}' '{print $10}'|awk -F, '{print $2}'`
		echo ${CYAN}NODE_IP: ${RESTORE} ${YELLOW}${NODE_IP}${RESTORE} 
		echo ${CYAN}RUNNING_NODES: ${RESTORE} ${YELLOW}${RUNNING_NODES}${RESTORE} ${CYAN}PARTITION: ${RESTORE} ${YELLOW}${PARTITION}${RESTORE}
		echo '-------------'
	else
		echo ${RED}ERROR: ${NODE_IP}_NODE_IS_FAILED.${RESTORE}
		echo '-------------'
	fi
done

#================================================
echo
echo "5.2 check rabbitmq sockets."
DIR5_2=${DIR}/rabbitmq-sockets.txt
# 自定义巡检节点数量
RABBITMQ_NUM=14
# txt文件中执行失败的IP
FAILED_LIST=(`grep -E 'UNREACHABLE|FAILED' ${DIR5_2}|awk '{print $1}'`)
# txt文件中执行成功的IP
SUCCESS_LIST=(`grep -E 'SUCCESS' ${DIR5_2}|awk '{print $1}'`)
i=0
for IP in ${SUCCESS_LIST[@]}
do
	# 获取sockets_limit和sockets_used的值
	SOCKETS_NUM=(`grep -E 'SUCCESS|sockets' ${DIR5_2}|grep -A2 ${IP}|grep -v ${IP}|awk -F'{|}|,' '{print $3}'`)
	if [ ${#SOCKETS_NUM[@]} -eq 2 ];then
		if [ $(echo $[${SOCKETS_NUM[0]} < ${SOCKETS_NUM[1]}]) -eq 1 ];then
			FAILED_SOCKET[$i]=${IP}
		else
			SUCCESS_SOCKET[$i]=${IP}
		fi
	else
		FAILED_SOCKET[$i]=${IP}
	fi
	let i++
done

if [ ${#FAILED_LIST[@]} -ne 0 ] || [ ${#FAILED_SOCKET[@]} -ne 0 ];then
	echo ${CYAN}RABBITMQ_NUM: ${RABBITMQ_NUM}${RESTORE} ${YELLOW}CHECK_NUM: $[${#FAILED_LIST[@]}+${#SUCCESS_LIST[@]}] ${RESTORE} ${GREEN}SUCCESS: ${#SUCCESS_SOCKET[@]}${RESTORE} ${RED}FAILED:  $[${#FAILED_LIST[@]}+${#FAILED_LIST[@]}+${#FAILED_SOCKET[@]}]${RESTORE} IP: ${RED}${FAILED_LIST[@]} ${FAILED_SOCKET[@]}${RESTORE}
else
	echo ${CYAN}RABBITMQ_NUM: ${RABBITMQ_NUM}${RESTORE} ${YELLOW}CHECK_NUM: $[${#FAILED_LIST[@]}+${#SUCCESS_LIST[@]}] ${RESTORE} ${GREEN}SUCCESS: ${#SUCCESS_SOCKET[@]}${RESTORE} '>>>>>' ${GREEN}RABBITMQ SOCKETS IS OK.${RESTORE}
fi

#================================================
echo
echo "6.1 check ceph cluster."
for i in {2..3}
do
DIR6_1=${DIR}/ceph-region$i-cluster.txt
# 成功执行节点数量
SUCCESS_NUM=`grep -Ec 'SUCCESS' ${DIR6_1}`
# 执行节点IP
NODE_IP=`grep -E '^([0-9]{1,3}\.){3}[0-9]{1,3}' ${DIR6_1}|awk '{print $1}'`

if [ ${SUCCESS_NUM} -eq 1 ];then
	CLUSTER_NAME=`grep -E '^[[:space:]]+cluster' ${DIR6_1}|awk '{print $2}'`
	CLUSTER_HEALTH=`grep -E '^[[:space:]]+health' ${DIR6_1}|awk '{print $2}'`
	MON_NUM=`grep -E '^[[:space:]]+monmap' ${DIR6_1}|awk '{print $3}'`
	OSD_NUM=`grep -E '^[[:space:]]+osdmap' ${DIR6_1}|awk '{print $3}'`
	OSD_UP=`grep -E '^[[:space:]]+osdmap' ${DIR6_1}|awk '{print $5}'`
	OSD_IN=`grep -E '^[[:space:]]+osdmap' ${DIR6_1}|awk '{print $7}'`
	VMS_PERC=`grep -E '^[[:space:]]+vms' ${DIR6_1}|awk '{print $4}'`
	VOLUMES_PERC=`grep -E '^[[:space:]]+volumes' ${DIR6_1}|awk '{print $4}'`
	IMAGES_PERC=`grep -E '^[[:space:]]+images' ${DIR6_1}|awk '{print $4}'`

	echo ${CYAN}NODE_IP: ${RESTORE} ${YELLOW}${NODE_IP}${RESTORE} 
	echo ${CYAN}CLUSTER_ID: ${RESTORE} ${YELLOW}${CLUSTER_NAME}${RESTORE}
	echo ${CYAN}CLUSTER_HEALTH: ${RESTORE} ${YELLOW}${CLUSTER_HEALTH}${RESTORE}
	echo ${CYAN}MON_NUM: ${RESTORE} ${YELLOW}${MON_NUM}${RESTORE}
	echo ${CYAN}OSD_NUM: ${RESTORE} ${YELLOW}${OSD_NUM}${RESTORE}
	echo ${CYAN}OSD_UP: ${RESTORE} ${YELLOW}${OSD_UP}${RESTORE}
	echo ${CYAN}OSD_IN: ${RESTORE} ${YELLOW}${OSD_IN}${RESTORE}
	echo ${CYAN}VMS_PERC: ${RESTORE} ${YELLOW}${VMS_PERC}%${RESTORE}
	echo ${CYAN}VOLUMES_PERC: ${RESTORE} ${YELLOW}${VOLUMES_PERC}%${RESTORE}
	echo ${CYAN}IMAGES_PERC: ${RESTORE} ${YELLOW}${IMAGES_PERC}%${RESTORE}
	echo '-------------'
else
	echo ${RED}ERROR: ${NODE_IP}_NODE_IS_FAILED.${RESTORE}
	echo '-------------'
fi
done
```

