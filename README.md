# kubernetes-ha-binary

## 项目介绍
项目致力于让有意向使用原生kubernetes集群的企业或个人，可以方便的、系统的使用**二进制**的方式手工搭建kubernetes高可用集群。并且让相关的人员可以更好的理解kubernetes集群的运作机制。

## 软件版本
- os centos7.2（ubuntu也适用，需要替换部分命令）
- kubernetes v1.16.4
- etcd v3.3.18
- docker 17.09.1-ce
- calico v3.10.2

软件兼容性参考:

https://github.com/coredns/deployment/blob/master/kubernetes/CoreDNS-k8s_version.md
https://docs.projectcalico.org/v3.10/getting-started/kubernetes/requirements

## 安装教程
#### [一、实践环境准备][1]
#### [二、高可用集群部署][2]
#### [三、集群可用性测试][3]
#### [四、部署ingress][4]
#### [五、部署metrics-server][5]
#### [六、部署dashboard][6]
#### [七、部署ingress-zookeeper][7]



[1]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/1-prepare.md
[2]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/2-ha-deploy.md
[3]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/3-test.md
[4]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/4-ingress-nginx.md
[5]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/5-metrics-server.md
[6]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/6-dashboard.md
[7]:https://github.com/Farmerddd/kubernetes-ha-binary/blob/master/docs/7-ingress-zookeeper.md
