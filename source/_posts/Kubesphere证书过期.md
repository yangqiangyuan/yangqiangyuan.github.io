---
title: Kubesphere证书过期
date: 2022-08-22 11:20:56
tags: Kubesphere
---

# Kubesphere证书过期

### 概述

 因为kubesphere的证书有效期只有一年，所以需要重新生成证书。

<!--more-->

### 1 kubesphere证书过期现象

```perl
# 使用kubectl命令的时候会提示证书过期
(DEV)[root@master01 ~]# kubectl get pods -n A
Unable to connect to the server: x509: certificate has expired or is not yet valid

# 检查证书有效期
(DEV)[root@master01 ~]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[check-expiration] Error reading configuration from the Cluster. Falling back to default configuration

W1228 10:09:37.193925    5469 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
CERTIFICATE                         EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                          Dec 27, 2021 10:22 UTC   <invalid>                               no      
apiserver                           Dec 27, 2021 10:22 UTC   <invalid>       ca                      no      
!MISSING! apiserver-etcd-client                                                                      
apiserver-kubelet-client            Dec 27, 2021 10:22 UTC   <invalid>       ca                      no      
controller-manager.conf             Dec 27, 2021 10:22 UTC   <invalid>                               no      
!MISSING! etcd-healthcheck-client                                                                    
!MISSING! etcd-peer                                                                                  
!MISSING! etcd-server                                                                                
front-proxy-client                  Dec 27, 2021 10:22 UTC   <invalid>       front-proxy-ca          no      
scheduler.conf                      Dec 27, 2021 10:22 UTC   <invalid>                               no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 25, 2030 10:22 UTC   8y              no      
!MISSING! etcd-ca                                                
front-proxy-ca          Dec 25, 2030 10:22 UTC   8y              no
```

### 2 解决方法

```perl
# 1.重新生成证书
# 在master01上生成新的证书
(DEV)[root@master01 ~]# kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[renew] Error reading configuration from the Cluster. Falling back to default configuration

W1228 10:12:47.177137   10207 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
MISSING! certificate the apiserver uses to access etcd
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
MISSING! certificate for liveness probes to healthcheck etcd
MISSING! certificate for etcd nodes to communicate with each other
MISSING! certificate for serving etcd
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
(DEV)[root@master01 ~]# 

# 2.将证书传到另外两个节点
# master02和master03先备份
(DEV)[root@master02 etc]# mv /etc/kubernetes /etc/kubernetes-bak-12-28
(DEV)[root@master03 etc]# mv /etc/kubernetes /etc/kubernetes-bak-12-28

# 将新生成的证书传到master02和master03
(DEV)[root@master01 etc]# scp -r kubernetes root@master02:/etc/
(DEV)[root@master01 etc]# scp -r kubernetes root@master03:/etc/

# 3.重启ETCD
# 登录master01和master02以及master03
(DEV)[root@master01 etc]# docker ps|grep etcd
f306a918736b   28c771d7cfbf                           "/usr/local/bin/etcd"    8 months ago   Up 8 months             etcd1
(DEV)[root@master01 etc]# docker restart f306a918736b
f306a918736b

(DEV)[root@master02 etc]# docker ps |grep etcd
11876daa3de5   30.23.8.56:5000/kubesphere/etcd:v3.3.12                            "/usr/local/bin/etcd"    8 months ago         Up 8 months                   etcd2
(DEV)[root@master02 etc]# docker restart 11876daa3de5
11876daa3de5

(DEV)[root@master03 etc]# docker ps|grep etcd
3a9ad934353b   30.23.8.56:5000/kubesphere/etcd:v3.3.12                 "/usr/local/bin/etcd"    7 seconds ago        Restarting (2) Less than a second ago             etcd3
(DEV)[root@master03 etc]# docker restart 3a9ad934353b
3a9ad934353b

# 4.生成新的config文件
# 不进行替换的话，kubectl的命令还是使用不了会提示如下错误。
(DEV)[root@master01 ~]# kubectl get pods -n hbp
Unable to connect to the server: x509: certificate has expired or is not yet valid

# 登录master01和master02以及master03
(DEV)[root@master01 .kube]# mv ~/.kube/config ~/.kube/config-bak-2021-12-28
(DEV)[root@master01 .kube]# cp /etc/kubernetes/admin.conf ~/.kube/config

(DEV)[root@master02 .kube]# mv ~/.kube/config ~/.kube/config-bak-2021-12-28
(DEV)[root@master02 .kube]# cp /etc/kubernetes/admin.conf ~/.kube/config

(DEV)[root@master03 etc]# mv ~/.kube/config ~/.kube/config-bak-2021-12-28
(DEV)[root@master03 etc]# cp /etc/kubernetes/admin.conf ~/.kube/config
```

