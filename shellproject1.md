# my-newoutsidedisk-1
linux basic courses
#!/bin/bash

# svc.sh start sshd
# -> systemctl enable sshd
# -> systemctl restart sshd
# -> systemctl status sshd
# svc.sh stop sshd
# -> systemctl disable sshd
# -> systemctl stop sshd
# -> systemctl status sshd

if [ $# -ne 2 ]; then
    echo "Usage: $0 <start|stop> <service>"
    exit 1
fi
ACTION=$1
SVC=$2

SVC_START() {
    systemctl enable $SVC >/dev/null 2>&1
    if [ $? -eq 0 ] ; then
        systemctl restart $SVC
        systemctl status $SVC
    else
        echo "[ FAIL ] 서비스 이름을 알수 없습니다."
        exit 3
    fi
}

SVC_STOP() {
    :
}

ERR_1() {
    :
}

case $ACTION in
    start) SVC_START ;;
    stop)  SVC_STOP ;;
    *) ERR_1 ; exit 2;;
esac
#!/bin/bash

# svc.sh start sshd
# -> systemctl enable sshd
# -> systemctl restart sshd
# -> systemctl status sshd
# svc.sh stop sshd
# -> systemctl disable sshd
# -> systemctl stop sshd
# -> systemctl status sshd

if [ $# -ne 2 ]; then
    echo "Usage: $0 <start|stop> <service>"
    exit 1
fi
SCRIPTNAME=$0
ACTION=$1
SVC=$2

SVC_START() {
    systemctl enable $SVC >/dev/null 2>&1
    if [ $? -eq 0 ] ; then
        systemctl restart $SVC
        systemctl status $SVC
    else
        echo "[ FAIL ] 서비스 이름을 알수 없습니다."
        exit 3
    fi
}

SVC_STOP() {
    systemctl disable $SVC >/dev/null 2>&1
    if [ $? -eq 0 ] ; then
        systemctl stop $SVC
        systemctl status $SVC
    else
        echo "[ FAIL ] 서비스 이름을 알수 없습니다."
        exit 4
    fi
}

ERR_1() {
    echo "Usage: $SCRIPTNAME <start|stop> <service>"
    exit 5
}

case $ACTION in
    start) SVC_START ;;
    stop)  SVC_STOP ;;
    *) ERR_1 ; exit 2;;
esac
#!/bin/bash

PASSWD=/etc/passwd

Menu() {
cat << EOF
(관리 목록)
------------------------------------
1) 사용자 추가
2) 사용자 확인
3) 사용자 삭제
4) 종료
------------------------------------
EOF
}

UserAdd() {
    :
}

UserVerify() {
cat << EOF
(사용자 확인)    
------------------------------------
$(awk -F: '$3 >= 1000 && $3 <= 60000 {print $1}' $PASSWD | cat -n)
------------------------------------

EOF
}

UserDel() {
    :
}

while true
do
    Menu
    echo -n "선택 번호? (1|2|3|4) : "
    read NUM

    case $NUM in
        1) UserAdd ;;
        2) UserVerify ;;
        3) UserDel ;;
        4) break ;;
        *) echo "[ WARN] 잘못된 선택을 했습니다." ; echo
    esac
done
#!/bin/bash
#   # ./check_service.sh 192.168.10.10 192.168.10.20
#   ------------------
#   192.168.10.10
#   ------------------
#   svc1    active
#   svc2    active
#
#   ------------------
#   192.168.10.20
#   ------------------
#   svc3    active
#   svc4    active
#   ....

if [ $# -ne 2 ] ; then
    echo "Usage: $0 <IP1> <IP2>"
    exit 1
fi
export FIRST_IP=$1
export SECOND_IP=$2
export BASEDIR=/root/bin

ServiceList() {
# input: str        # ServiceList main
# output: file      # ServiceList main -> main.txt
# functions:  
SERVER=$1   
ssh $SERVER systemctl -t service | sed -n '1,/^LOAD/p' \
                                 | sed '$d' \
                                 | awk '{print $1, $3}' > $BASEDIR/$SERVER.txt
}

ServiceList main
#!/bin/bash
#   # ./check_service.sh 192.168.10.10 192.168.10.20
#   ------------------
#   192.168.10.10
#   ------------------
#   svc1    active
#   svc2    active
#
#   ------------------
#   192.168.10.20
#   ------------------
#   svc3    active
#   svc4    active
#   ....

if [ $# -ne 2 ] ; then
    echo "Usage: $0 <IP1> <IP2>"
    exit 1
fi
export FIRST_IP=$1
export SECOND_IP=$2
export BASEDIR=/root/bin

ServiceList() {
# input: str        # ServiceList main
# output: file      # ServiceList main -> main.txt
# functions:  
SERVER=$1   
ssh $SERVER systemctl -t service | sed -n '1,/^LOAD/p' \
                                 | sed '$d' \
                                 | awk '{print $1, $3}' > $BASEDIR/$SERVER.txt
}

ServiceList $FIRST_IP
ServiceList $SECOND_IP

FSERVER=$BASEDIR/$FIRST_IP.txt
SSERVER=$BASEDIR/$SECOND_IP.txt

cat << EOF
###########################
"$FIRST_IP"
###########################
$(diff $FSERVER $SSERVER | fgrep '<' | cut -c3-)

###########################
"$SECOND_IP"
###########################
$(diff $FSERVER $SSERVER | fgrep '>' | cut -c3-)

EOF
#!/bin/bash

PID1=$(ps -ef | grep -w code \
                        | grep -v 'grep --color' \
                        | awk '{print $2}' \
                        | sort -n \
                        | head -1)

[ -n "$PID1" ] && kill $PID1
#!/bin/bash

TMP1=/tmp/tmp1
TMP2=/tmp/tmp2

# 1. PV
# 2. VG
# 3. LV

############################
# 1. PV
############################
# (1) View
fdisk -l | grep LVM | awk '{print $1}' > $TMP1
pvs | tail -n +2 | awk '{print $1}' > $TMP2

cat << EOF
############# PV VIEW #############
$(cat $TMP1 $TMP2 | sort | uniq -u)
###################################

EOF

# (2) Works
echo "=> 위의 목록에서 PV로 생성하고 싶은 볼륨을 선택합니다. <="
echo -n "볼륨 선택? (ex: /dev/sdb1 /dev/sdc1 ...) : "
read VOLUME

pvcreate $VOLUME >/dev/null 2>&1
if [ $? -eq 0 ] ; then
    echo "[  OK  ] Physical volume $VOLUME successfully created."
    pvs
else
    echo "[ FAIL ] Physical volume not created."
    exit 1 
fi

############################
# 2. VG
############################
# (1) View
# (2) Works
############################
# 3. LV
############################
# (1) View
# (2) Works

#!/bin/bash

#
# 변수 선언
#
TMP1=/tmp/tmp1
TMP2=/tmp/tmp2
TMP3=/tmp/tmp3
TMP4=/tmp/tmp4

#
# PV 생성
#
while true
do
echo ; echo "####################### (1) PV 생성 ###########################" ; echo
# (1) PV 생성 가능한 목록
fdisk -l | grep LVM | awk '{print $1}' > $TMP1
pvs | tail -n +2 | awk '{print $1}' > $TMP2

echo "############# 사용 가능한 PV 목록 ###############"
echo "아래 목록에서 PV로 생성하고 싶은 볼륨을 선택합니다."
echo "$(cat $TMP1 $TMP2 | sort | uniq -u)"
echo "################################################"

# (2) PV 생성 작업
echo -n "볼륨 선택? (ex: /dev/sdb1 /dev/sdc1 ...) : "
read VOLUME

cat << EOF

    다음 명령어를 정말 실행하시겠습니까?
    # pvcreate $VOLUME
    * yes: 실행, no: 처음부터 다시, skip: 작업 뛰어넘기

EOF
echo -n "당신의 선택은? (yes|no|skip) : "
read CHOICE
case $CHOICE in
    yes) : ;;
    no) continue ;;
    skip) break ;;
    *) continue ;;
esac

pvcreate "$VOLUME" >/dev/null 2>&1
if [ $? -eq 0 ] ; then
    echo "[  OK  ] Physical volume $VOLUME successfully created."
    echo "_______________________________________________________"    
    pvs
else
    echo "[ FAIL ] Physical volume not created."
    exit 1 
fi
break
done

#
# VG 생성
#
while true
do
echo ; echo "####################### (2) VG 생성 ###########################" ; echo
# (1) VG 포함할 수 있는 PV 목록
vgs | tail -n +2 | awk '{print $1}' > $TMP3
pvs > $TMP4
for i in $(cat $TMP3)
do
    sed -i "/$i/d" $TMP4
done

echo "################# VG에 포함 가능한 PV 목록 #################"
echo "$(cat $TMP4)"
echo "############################################################"

# (2) VG 생성 과정
# vgcreate vg1 /dev/sdb1 /dev/sdc1
echo -n "VG 이름은? (ex: vg1) : "
read VGNAME

echo -n "VG에 넣을 PV 목록은? (ex: /dev/sdb1 /dev/sdc1 ...) : "
read PVLIST

cat << EOF

    다음 명령어를 정말 실행하시겠습니까?
    # vgcreate $VGNAME $PVLIST
    * yes: 실행, no: 처음부터 다시, skip: 작업 뛰어넘기, exit: 프로그램 종료

EOF
echo -n "당신의 선택은? (yes|no|skip|exit) : "
read CHOICE
case $CHOICE in
    yes) :       ;;
    no) continue ;;
    skip) break  ;;
    exit) exit 8 ;;
    *) continue  ;;
esac

vgcreate "$VGNAME" "$PVLIST" >/dev/null 2>&1
if [ $? -eq 0 ] ; then
    echo "[  OK  ] Volume group $VGNAME successfully created"
    echo "__________________________________________________"
    vgs
else
    echo "[ FAIL ] Volume group not created."
    exit 2
fi
break
done

#
# LV 생성
#
while true
do
echo ; echo "####################### (3) LV 생성 ###########################" ; echo
# (1) VG에 남은 공간 확인
echo "################# 사용 가능한 VG 목록 #################"
echo "$(vgs | awk '$7 != '0' {print $0}')"
echo "########################################################"

# (2) LV 생성 과정
# lvcreate vg1 -n lv1 -L 500m
echo -n "LV를 생성할 VG 이름은? (ex: vg1) : "
read VGLV

echo -n "생성할 LV 이름은? (ex: lv1) : "
read LVNAME

echo -n "LV 용량은? (ex: 500m) : "
read LVSIZE

cat << EOF

    다음 명령어를 정말 실행하시겠습니까?
    # lvcreate $VGLV -n $LVNAME -L $LVSIZE
    * yes: 실행, no: 처음부터 다시, skip: 작업 뛰어넘기, exit: 프로그램 종료

EOF
echo -n "당신의 선택은? (yes|no|skip|exit) : "
read CHOICE
case $CHOICE in
    yes) : ;;
    no) continue ;;
    skip) break  ;;
    exit) exit 9 ;;
    *) continue  ;;
esac

lvcreate "$VGLV" -n "$LVNAME" -L "$LVSIZE" >/dev/null 2>&1
if [ $? -eq 0 ] ; then
    echo "[  OK  ] Logical Volume $LVNAME successfully created."
    echo "_________________________________________________________________________________"
    lvs
else
    echo "[ FAIL ] Logical Volume not created."
    exit 3
fi
break
done
#!/bin/bash

#    # crontab -e
#    Min Hour Day Mon Week CMD
#     0   8    *   *   *   /root/shell/check_file.sh
#
#    # cat /root/shell/test/check_file_list.txt
#    /etc/passwd
#    /etc/group
#    /etc/hosts
#    .....

F_LIST=/root/bin/file_list.txt                      # 점검할 파일 목록
T_FILE=/tmp/tmp1                                    # 임시파일
T_FILE2=/tmp/tmp2
T_FILE3=/tmp/tmp3
F_RESULT=/root/bin/result.$(date +'%m%d')           # 결과 리포트 파일
EMAIL=root                                          # 결과를 받을 이메일 주소
BACKUPDIR=/backup

> $F_RESULT


for F_NAME in $(cat $F_LIST)                            # 점검해야 하는 파일 목록 읽기
do
    # ==> echo $F_NAME ; read       # F_NAME=/root/bin/passwd
    DIRNAME=$(dirname $F_NAME)      # /root/bin
    FILENAME=$(basename $F_NAME)    # passwd
    BACKUPFILE=$BACKUPDIR/$FILENAME.orig                # BACKUPFILE=/backup/passwd.orig
    # ==> echo "$DIRNAME $FILENAME $BACKUPFILE" ; read
    if [ -f "$BACKUPFILE" ] ; then                      # 백업파일 존재 유무 확인
        diff $F_NAME $BACKUPFILE > $T_FILE              # 원본파일<-->백업파일 비교
        if [ -s $T_FILE ] ; then
            echo "[ WARN ] $F_NAME" >> $F_RESULT
            /bin/cp $F_NAME $BACKUPFILE.$(date +%m%d_%H%M%S)
            /bin/cp $F_NAME $BACKUPFILE
        else
            echo "[  OK  ] $F_NAME" >> $F_RESULT
        fi
    else
        /bin/cp $F_NAME $BACKUPFILE
    fi
    # ==> read  
done

grep -qw WARN $F_RESULT
if [ $? -eq 0 ] ; then
    mailx -s "[ WARN ] Check Files" $EMAIL < $F_RESULT
else
    mailx -s "[  OK  ] Have a good time." $EMAIL < $F_RESULT
fi

# rm T_FILE                     # tempory file delete

#
# Sfecific configuration
#
export PS1='\[\e[31;1m\][\u@\h\[\e[33;1m\] \w]\$ \[\e[m\]'
alias lsf='ls -l | grep "^-"'
alias lsd='ls -l | grep "^d"'
alias ls='ls --color=auto -h -F'
alias c='clear'
alias cear='clear'
alias clar='clear'
alias pps='ps -ef | head -1 ; ps -ef | grep $1'
alias nstate='netstat -antup | head -2 ; netstat -antup | grep $1'
alias tree='env LANG=C tree'
alias vi='/usr/bin/vim'
alias grep='grep --color=auto -i'
alias df='df -h -T'
alias chrome='/usr/bin/google-chrome --no-sandbox'
alias RS='rsync -avz --delete -e ssh'
alias LS='rsync -av --delete'
alias fwcmd='firewall-cmd'
alias fwlist='firewall-cmd --list-all'
alias fwreload='firewall-cmd --reload'
rpm --import https://packages.microsoft.com/keys/microsoft.asc

cat << EOF > /etc/yum.repos.d/vscode.repo
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc" 
EOF

yum install code

#!/bin/bash

TMP1=/tmp/tmp1

export LANG=en_US.UTF-8
DATE=$(date +'%Y.%m.%d %H:%M:%S')
# echo -n "Enter your name : "
# read NAME
NAME="Baik SeoungChan"
OS=$(cat /etc/centos-release)
KERNEL=$(uname -sr)
CPUS=$(lscpu | grep '^CPU(s):' | awk '{print $2}')
MEM=$(free -h | grep '^Mem:' | awk '{print $2}')
DISKS=$(lsblk -S | awk '$3 == "disk" {print $0}' | wc -l)

cat << EOF

Server Vul. Checker version 1.0
DATE: $DATE
NAME: $NAME

(1) Server Information
============================================
OS          : $OS
Kernel      : $KERNEL
CPU         : $CPUS
MEM         : $MEM
DISK        : $DISKS
EOF

nmcli connection | grep -vw ' -- ' | tail -n +2 | awk '{print $4}' > $TMP1
DNS=$(for i in $(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
do
    echo -n "$i "
done)

SEQ=$(nmcli connection | grep -vw ' -- ' | tail -n +2 | wc -l)
for i in $(seq $SEQ)
do
    NIC=$(sed -n "${i}p" $TMP1)
    cat << EOF
Network ${i} ($NIC):
    * IP: $(ifconfig $NIC | grep 'inet ' | awk '{print $2}')
    * Netmask: $(ifconfig $NIC | grep 'inet ' | awk '{print $4}')
EOF
done
echo "  * Defaultrouter: $(ip route | grep default | awk '{print $3}')"
echo "  * DNS: $DNS"
echo 


