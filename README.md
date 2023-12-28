
观测云演示数据

 |K8S|微服务|Pod|Controller|
|:-:|:-:|:-:|:-:|
|浏览器日志|SourceMap|IP解析|会话回放|
|VIP跟踪|eBPF|集群事件|业务日志|
|日志字段治理|APM|Profiling|Mysql|

部署架构图

<div align=left><img src="https://github.com/zhaogangxp/GuanceCloud/assets/28213758/7de0c14d-d357-4d2f-a42c-131f4e0a4923" style="width: 30%;"></div>

资源准备

|资源|规格要求|
|:-:|:-:|
|操作系统|CentOS 7.9 Minimal安装|
|Demo主机 |2C 8G 20GB 物理机或云主机1台|
|Demo主机|可访问到观测云SaaS平台|
|电脑|可访问到Demo主机|

注意事项

* 必须该文的ingress控制器，因Demo应用的域名已经绑定此类控制器，并且ingress控制器节点充当网络访问入口。

* 必须按照部署步骤，顺序执行，因为服务间有一定的调用依赖关系，如：nacos调用mysql，必须等mysql启动并初始化完成。各服务模块（auth、gateway、system）必须等待nacos启动完成，以便完成微服务的注册。

* 域名必须使用如下访问（本地客户端指定HOSTS文件强制解析如下域名）：

  ruoyi.dataflux.cn Demo系统的应用（SpringCloud）

  datakit.dataflux.cn 观测云采集器datakit的Rum数据接收地址

部署步骤

1 部署Kubernetes及其Ingress控制器

`下载sealos至Demo主机上，执行如下命令安装集群并部署ingress`

[Sealos4.3.6](https://github.com/labring/sealos/releases/tag/v4.3.6 "Sealos")

```
modprobe br_netfilter
tar zxvf sealos_4.3.6_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin
sealos run labring/kubernetes:v1.24.0 labring/helm:v3.8.2 labring/calico:v3.22.1 labring/metrics-server:v0.6.1 --single
kubectl apply -f ingress.yaml
```

2 在访问Demo的电脑上添加hosts解析如下

```
192.168.167.160 ruoyi.dataflux.cn
192.168.167.160 datakit.dataflux.cn
```
3 创建 应用ID 在观测云控制台-用户访问监测-新建应用

`appid_1ae7755342fb47f0a7bcd62830d557c8`

<div align=left><img src="https://github.com/zhaogangxp/gc/assets/28213758/c66bc3a7-b9c3-4b76-8dfb-eb7de6880010" style="width: 30%;"></div>

4 部署观测云采集器

<font color=#ff0000>修改datakit.yaml中的token为你的观测云工作空间看到的token</font>

`kubectl apply -f datakit.yaml`

5 部署Demo应用

```
kubectl create namespace ruoyi
kubectl apply -f redis.yaml
kubectl apply -f mysql.yaml
```
```
验证mysql启动完成
kubectl logs -n ruoyi mysql-pod-name

Version: '5.7.39'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```
```
kubectl apply -f nacos.yaml
```
```
验证nacos启动完成
kubectl logs -n ruoyi nacos-pod-name

2023-12-28 22:56:35,922 INFO Nacos started successfully in stand alone mode. use external storage
```

```
kubectl apply -f auth.yaml
kubectl apply -f gateway.yaml
kubectl apply -f system.yaml
kubectl apply -f web.yaml
```
```
验证auth gateway system启动完成
kubectl logs -n ruoyi auth/gateway/system-pod-name

 .-------.       ____     __        
 |  _ _   \      \   \   /  /    
 | ( ' )  |       \  _. /  '       
 |(_ o _) /        _( )_ .'         
 | (_,_).' __  ___(_ o _)'          
 |  |\ \  |  ||   |(_,_)'         
 |  | \ `'   /|   `-'  /           
 |  |  \    /  \      /           
 ''-'   `'-'    `-..-'  
```

6 访问Demo应用

浏览器打开域名 http://ruoyi.dataflux.cn

![image](https://github.com/zhaogangxp/GuanceCloud/assets/28213758/e006b884-e218-46d8-9430-bbee3c139cab)
