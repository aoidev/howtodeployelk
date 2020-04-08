# howtodeployelk
利用K8S部署ELK服务（v7.2.0）
前提是已经部署好k8s，我这里只部署了单节点


`es-master`索引文件存放在`/var/lib/docker/data/es/master`目录下，以pvc挂载至容器内部`/usr/share/elasticsearch/data`目录，启动命令及es配置参数在YAML文件中进行配置。
## 1、先把本项目中的yml文件clone下去
## 2
```bash
# 创建物理文件夹
mkdir /var/lib/docker/data/es/master
```

## 3、创建namespace
```bash
kubectl apply -f ns.yml
```

## 4、创建es服务
```bash
kubectl apply -f es-service.yml
kubectl apply -f es-master.yml
```


## 5、部署Kibana
```bash
# 创建kibana服务
kubectl apply -f kibana.yml

# 验证kibana服务
curl -s http://localhost:25601
```

## 6、部署Logstash

logstash配置文件定义在ConfigMap中。

```bash
# 创建logstash服务
kubectl apply -f logstash-named-k8s.yml
```
# howtodeployelk
