Alpine基础镜像

#先添加阿里源
cat > /etc/apk/repositories << EOF
http://mirrors.aliyun.com/alpine/v3.12/main/
http://mirrors.aliyun.com/alpine/v3.12/community
EOF

 

Alpine镜像中的telnet在3.7版本后被转移至busybox-extras包中，需要使用apk单独安装
#安装 curl scp telnet vim
apk update && apk add curl openssh-client busybox-extras vim