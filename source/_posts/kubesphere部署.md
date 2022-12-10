---
title: kubesphere部署
date: 2022-11-25 10:39:06
tags: kubesphere
---

# 青云部署k8s

获取离线安装包

<!--more-->

```
青云部署k8s
获取离线安装包

curl -Ok https://kubesphere-installer.pek3b.qingstor.com/offline/v3.0.0/kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz


解压离线安装包
tar-xvf kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz -C /usr/local/

集群安装

1.创建集群配置文件

./kk create config --with-kubesphere v3.0.0

2.修改集群配置文件
vi config-sample.yaml
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: laopai-uat
spec:
  hosts:
  - {name: master01, address: 192.168.0.183, internalAddress: 192.168.0.183, user: root, password: password}
  - {name: master02, address: 192.168.0.184, internalAddress: 192.168.0.184, user: root, password: password}
  - {name: master03, address: 192.168.0.185, internalAddress: 192.168.0.185, user: root, password: password}
  - {name: node01, address: 192.168.0.186, internalAddress: 192.168.0.186, user: root, password: password}
  - {name: node02, address: 192.168.0.187, internalAddress: 192.168.0.187, user: root, password: password}
  - {name: node03, address: 192.168.0.188, internalAddress: 192.168.0.188, user: root, password: password}
  - {name: node04, address: 192.168.0.189, internalAddress: 192.168.0.189, user: root, password: password}
  - {name: node05, address: 192.168.0.190, internalAddress: 192.168.0.190, user: root, password: password}
  roleGroups:
    etcd:
    - master01
    - master02
    - master03
    master: 
    - master01
    - master02
    - master03
    worker:
    - node01
    - node02
    - node03
    - node04
    - node05
  controlPlaneEndpoint:
    domain: kubesphere.com
    address: "192.168.0.183"
    port: "6443"
  kubernetes:
    version: v1.18.6
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: ["192.168.0.183.2:5000"]
    privateRegistry: 192.168.0.183.2:5000
  addons: 
  - name: nfs-client
    namespace: kube-system
    sources: 
      chart:
        name: nfs-client-provisioner
        #repo: https://charts.kubesphere.io/main
        path: /data1/kubesphere-all-v3.0.0-offline-linux-amd64/charts
        values:
        - image.repository=192.168.0.183.2:5000/kubesphere/nfs-client-provisioner
        - nfs.server=100.68.212.1
        - nfs.path=/data
        - storageClass.defaultClass=true


---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.0.0
spec:
  local_registry: ""
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  etcd:
    monitoring: true
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    es:
      elasticsearchDataVolumeSize: 20Gi
      elasticsearchMasterVolumeSize: 4Gi
      elkPrefix: logstash
      logMaxAge: 7
    mysqlVolumeSize: 20Gi
    minioVolumeSize: 20Gi
    etcdVolumeSize: 20Gi
    openldapVolumeSize: 2Gi
    redisVolumSize: 2Gi
  console:
    enableMultiLogin: false  # enable/disable multi login
    port: 30880
  alerting:
    enabled: true
  auditing:
    enabled: true
  devops:
    enabled: true
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsJavaOpts_MaxRAM: 2g
  events:
    enabled: true
    ruler:
      enabled: true
      replicas: 2
  logging:
    enabled: true
    logsidecarReplicas: 2
  metrics_server:
    enabled: true
  monitoring:
    prometheusMemoryRequest: 400Mi
    prometheusVolumeSize: 20Gi
  multicluster:
    clusterRole: none  # host | member | none
  networkpolicy:
    enabled: true
  notification:
    enabled: true
  openpitrix:
    enabled: true
  servicemesh:
    enabled: true

3.在镜像机器搭建registry私服

docker run -d --name registry2 --restart=always -v /data/registry:/var/lib/registry -p5000:5000registry:2



registry2安装后，集群所有节点需设置docker daemon.json文件insecure后重启docker

4.导入镜像

推送语句

./push-images.sh 192.168.0.183:5000

镜像清单

k8s：

kubesphere/kube-apiserver:v1.17.9

kubesphere/kube-scheduler:v1.17.9

kubesphere/kube-proxy:v1.17.9

kubesphere/kube-controller-manager:v1.17.9

kubesphere/pause:3.1

kubesphere/pause:3.2 (k8s版本大于v1.18)

kubesphere/etcd:v3.3.12

calico/kube-controllers:v3.15.1

calico/node:v3.15.1

calico/cni:v3.15.1

calico/pod2daemon-flexvol:v3.15.1

coredns/coredns:1.6.9

kubesphere/k8s-dns-node-cache:1.15.12



localVolume：

kubesphere/node-disk-manager:0.5.0

kubesphere/node-disk-operator:0.5.0

kubesphere/provisioner-localpv:1.10.0

kubesphere/linux-utils:1.10.0



kubesphere：

kubesphere/ks-apiserver:v3.0.0

kubesphere/ks-console:v3.0.0

kubesphere/ks-controller-manager:v3.0.0

kubesphere/ks-installer:v3.0.0

kubesphere/etcd:v3.2.18

kubesphere/kubectl:v1.0.0

kubesphere/ks-upgrade:v3.0.0

kubesphere/ks-devops:flyway-v3.0.0

redis:5.0.5-alpine

alpine:3.10.4

haproxy:2.0.4

mysql:8.0.11

nginx:1.14-alpine

minio/minio:RELEASE.2019-08-07T01-59-21Z

minio/mc:RELEASE.2019-08-07T23-14-43Z

mirrorgooglecontainers/defaultbackend-amd64:1.4

kubesphere/nginx-ingress-controller:0.24.1

osixia/openldap:1.3.0

csiplugin/snapshot-controller:v2.0.1

kubesphere/ks-upgrade:v3.0.0

kubesphere/ks-devops:flyway-v3.0.0



monitoring：

kubesphere/prometheus-config-reloader:v0.38.3

kubesphere/prometheus-operator:v0.38.3

prom/alertmanager:v0.21.0

prom/prometheus:v2.20.1

kubesphere/node-exporter:ks-v0.18.1

jimmidyson/configmap-reload:v0.3.0

kubesphere/notification-manager-operator:v0.1.0

kubesphere/notification-manager:v0.1.0

kubesphere/metrics-server:v0.3.7

kubesphere/kube-rbac-proxy:v0.4.1

kubesphere/kube-state-metrics:v1.9.6



logging:

kubesphere/elasticsearch-oss:6.7.0-1

kubesphere/fluentbit-operator:v0.2.0

kubesphere/fluentbit-operator:migrator

kubesphere/fluent-bit:v1.4.6

kubesphere/kube-auditing-operator:v0.1.0

kubesphere/kube-auditing-webhook:v0.1.0

kubesphere/kube-events-exporter:v0.1.0

kubesphere/kube-events-operator:v0.1.0

kubesphere/kube-events-ruler:v0.1.0

kubesphere/log-sidecar-injector:1.1

docker:19.03

service-mesh：

istio/citadel:1.4.8

istio/galley:1.4.8

istio/kubectl:1.4.8

istio/mixer:1.4.8

istio/pilot:1.4.8

istio/proxyv2:1.4.8

istio/sidecar_injector:1.4.8

jaegertracing/jaeger-agent:1.17

jaegertracing/jaeger-collector:1.17

jaegertracing/jaeger-operator:1.17.1

jaegertracing/jaeger-query:1.17

jaegertracing/jaeger-es-index-cleaner:1.17.1



devops：

jenkins/jenkins:2.176.2

jenkins/jnlp-slave:3.27-1

kubesphere/jenkins-uc:v3.0.0

kubesphere/s2ioperator:v2.1.1

kubesphere/s2irun:v2.1.1

kubesphere/builder-base:v2.1.0

kubesphere/builder-nodejs:v2.1.0

kubesphere/builder-maven:v2.1.0

kubesphere/builder-go:v2.1.0

kubesphere/s2i-binary:v2.1.0

kubesphere/tomcat85-java11-centos7:v2.1.0

kubesphere/tomcat85-java11-runtime:v2.1.0

kubesphere/tomcat85-java8-centos7:v2.1.0

kubesphere/tomcat85-java8-runtime:v2.1.0

kubesphere/java-11-centos7:v2.1.0

kubesphere/java-8-centos7:v2.1.0

kubesphere/java-8-runtime:v2.1.0

kubesphere/java-11-runtime:v2.1.0

kubesphere/nodejs-8-centos7:v2.1.0

kubesphere/nodejs-6-centos7:v2.1.0

kubesphere/nodejs-4-centos7:v2.1.0

kubesphere/python-36-centos7:v2.1.0

kubesphere/python-35-centos7:v2.1.0

kubesphere/python-34-centos7:v2.1.0

kubesphere/python-27-centos7:v2.1.0



notification&&alerting:

kubesphere/notification:flyway_v2.1.2

kubesphere/notification:v2.1.2

kubesphere/alert-adapter:v3.0.0

kubesphere/alerting-dbinit:v3.0.0

kubesphere/alerting:v2.1.2



multicluster:

kubesphere/kubefed:v0.3.0

kubesphere/tower:v0.1.0



openpitrix(app store)：

openpitrix/generate-kubeconfig:v0.5.0

openpitrix/openpitrix:flyway-v0.5.0

openpitrix/openpitrix:v0.5.0

openpitrix/release-app:v0.5.0

5.节点初始化

yum install -y socat conntrack ebtables ipset

./kk init os -f config-sample.yaml -s ./dependencies/

6.执行集群安装

集群安装语句

./kk create cluster -f config-sample.yaml

显示安装结果

#####################################################

###Welcome to KubeSphere!###

#####################################################



Console: http://192.168.0.8:30880

Account: admin

Password: P@88w0rd



NOTES：

1.After logging into theconsole, please check the

monitoring statusofservice componentsin

the"Cluster Management". If any serviceisnot

ready, please wait patientlyuntilall components

are ready.

2.Please modify thedefaultpassword after login.



#####################################################

https://kubesphere.io

20xx-xx-xx xx:xx:xx

#####################################################

检查安装

kubectl logs -n kubesphere-system$(kubectl get pod -n kubesphere-system-l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f

7.集群清理/卸载

./kk delete cluster –f config-sample.yaml

8.集群增加节点

编辑追加config-sample,host行

./kk add nodes -f config-sample.yaml

9.集群删除节点

./kk delete node nodeName -f config-sample.yaml
```

