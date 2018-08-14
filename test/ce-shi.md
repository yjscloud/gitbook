# 测试

## ==============================================

echo echo "6.2 check ceph log."

## 自定义巡检节点数量

LOG\_NODES=6 DIR6\_2=${DIR}/ceph-log.txt

## 有ERROR节点数量

FAILED=\(`grep -E 'SUCCESS' ${DIR6_2}|awk '{print $1}'`\)

## 无ERROR节点数量

NO\_ERR=\(`grep -E 'FAILED' ${DIR6_2}|awk '{print $1}'`\)

if \[ ${\#FAILED\[@\]} -eq 0 \];then echo ${CYAN}LOG\_NODES: ${LOG\_NODES}${RESTORE} ${YELLOW}CHECK\_NUM: $\[${\#FAILED\[@\]}+${\#NO\_ERR\[@\]}\] ${RESTORE} ${GREEN}SUCCESS: ${\#NO\_ERR\[@\]}${RESTORE} '&gt;&gt;&gt;&gt;&gt;' ${GREEN}CEPH LOG IS OK.${RESTORE} else echo ${CYAN}LOG\_NODES: ${LOG\_NODES}${RESTORE} ${YELLOW}CHECK\_NUM: $\[${\#FAILED\[@\]}+${\#NO\_ERR\[@\]}\] ${RESTORE} ${GREEN}SUCCESS: ${\#NO\_ERR\[@\]}${RESTORE} ${RED}FAILED: ${\#FAILED\[@\]}${RESTORE} IP: ${RED}${FAILED\[@\]}${RESTORE} fi

## =============================================

echo echo "7. check df capacity." DIR7=${DIR}/disk-capacity-percentage.txt

## 检查的服务器数量

CHECK\_NUM=`grep -Ec '^([0-9]{1,3}\.){3}[0-9]{1,3}' $DIR7`

## 容量超过90%的服务器数量

ERR\_NUM=`grep -Ev 'FAILED|UNREACHABLE|^[[:space:]]+|^}|^Filesystem' ${DIR7}|awk -F'%| +' '{if ($5 > 90) print $0}'|grep -E '^/dev/sd' -B1|grep -Ec '([0-9]{1,3}.){3}[0-9]{1,3}'`

if \[ ${ERR\_NUM} -eq 0 \];then echo ${YELLOW}CHECK\_NUM: ${CHECK\_NUM}${RESTORE} '&gt;&gt;&gt;&gt;&gt;' ${GREEN} All server disk capacity utilization is lower than 90%. ${RESTORE} else echo ${YELLOW}CHECK\_NUM: ${CHECK\_NUM}${RESTORE} ${RED} FAILED\_SERVER\_NUM: ${ERR\_NUM} ${RESTORE} echo ${RED} grep -Ev 'FAILED\|UNREACHABLE\|^\[\[:space:\]\]+\|^}\|^Filesystem' ${DIR7}\|awk -F'%\| +' '{if \($5 &gt; 90\) print $0}'\|grep -E '^/dev/sd' -B1 echo ${RESTORE} fi

## =============================================

echo echo "8. check create max 32G vm number more than 3."

## 各分区能创建大型虚拟机数量

DIR8=${DIR}/vm-max-result.txt echo ${YELLOW} grep -Ev 'sum' ${DIR8} \|awk '{print $1}'\|awk '{if\(NR%2==0\) ORS="\n";else ORS=" ---&gt; ";print}' echo ${RESTORE}

## ==============================================

echo "9. check openstack service." for i in {1..4} do DIR9=${DIR}/openstack-region$i if \[ -f ${DIR9} \];then SUCCESS=\(\) FAILED=\(\)

NOVA\_STATE=`grep 'nova-' ${DIR9}|grep -vc up` NEITRON\_STATE=`grep neutron ${DIR9}|grep -vc ':-)'` CINDER\_STATE=`grep 'cinder-' ${DIR9}|grep -c up` TENANT\_STATE=`grep -Ec '[[:space:]]admin[[:space:]]' ${DIR9}` GALNCE\_STATE=`grep -Ec 'bare' ${DIR9}`

if \[ ${NOVA\_STATE} -eq 0 \];then SUCCESS\[0\]='NOVA' else FAILED\[0\]='NOVA' fi if \[ ${NEITRON\_STATE} -eq 0 \];then SUCCESS\[1\]='NEUTRON' else FAILED\[1\]='NEUTRON' fi if \[ ${CINDER\_STATE} -eq 6 \];then SUCCESS\[2\]='CINDER' else FAILED\[2\]='CINDER' fi if \[ ${TENANT\_STATE} -eq 3 \];then SUCCESS\[3\]='TENANT' else FAILED\[3\]='TENANT' fi if \[ ${GALNCE\_STATE} -gt 3 \];then SUCCESS\[4\]='GLANCE' else FAILED\[4\]='GLANCE' fi

if \[ ${\#FAILED\[@\]} -eq 0 \];then echo ${CYAN}REGION: `basename ${DIR9}|tr '[a-z]' '[A-Z]'` ${RESTORE} ${GREEN}CHECK\_OK: ${SUCCESS\[@\]}${RESTORE} '&gt;&gt;&gt;&gt;&gt;' ${GREEN}`basename ${DIR9}|tr '[a-z]' '[A-Z]'` IS OK.${RESTORE} else echo ${CYAN}REGION: `basename ${DIR9}|tr '[a-z]' '[A-Z]'` ${RESTORE} ${YELLOW}CHECK\_OK: ${SUCCESS\[@\]}${RESTORE} ${RED}FAILED: ${FAILED\[@\]}${RESTORE} fi else echo ${RED}NO SUCH FILE: `basename ${DIR9}|tr '[a-z]' '[A-Z]'`${RESTORE} fi  
done

