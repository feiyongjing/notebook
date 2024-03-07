### 参考：https://minbaby.github.io/post/2022/10/k8s-config-ingress-nginx-default-https-certificate/#:~:text=k8s%20%E9%85%8D%E7%BD%AE%20ingress-nginx%20%E9%BB%98%E8%AE%A4%20https%20%E8%AF%81%E4%B9%A6%201%20%E4%BD%BF%E7%94%A8,describe%20-n%20ingress%20daemonset.apps%2Fnginx-ingress-microk8s-controller%20%7C%20grep%20--default-ssl-certificate%20
- 使用 https 证书文件生成 secret 命令是 kubectl create secret tls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE} -n kube-system
- 修改 ingress-nginx 的配置文件
- 查看 ingress 的名字 kubectl get all -A | grep ingress
- 查看 ingress-nginx 是否已经配置 kubectl describe -n ingress daemonset.apps/nginx-ingress-microk8s-controller | grep --default-ssl-certificate
- 编辑配置 kubectl edit -n ingress daemonset.apps/nginx-ingress-microk8s-controller
- 修改/增加 --default-ssl-certificate=kube-system/tls