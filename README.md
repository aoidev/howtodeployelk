# howtodeployelk
利用K8S部署ELK服务（v7.2.0）
前提是已经部署好k8s，我这里只部署了单节点


`es-master`索引文件存放在`/var/lib/docker/data/es/master`目录下，以pvc挂载至容器内部`/usr/share/elasticsearch/data`目录，启动命令及es配置参数在YAML文件中进行配置。
## 1、先把本项目中的yml文件clone下去


## 2、创建物理文件夹
```bash
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
```

## 6、部署Logstash

logstash配置文件定义在ConfigMap中。

```bash
# 创建logstash服务
kubectl apply -f logstash-named-k8s.yml
```
# howtodeployelk

# Tips
这里描述一下遇到的一些坑吧。
## 1、PersistentVolume和PersistentVolumeClaim的关系
    可以看到在es-master.yml这个文件中分别定义了上述两个对象，PersistentVolume可以理解为我们在服务器中申请的一块存储区域；
    PersistentVolumeClaim可以理解为：又向前面提到的PersistentVolume申请的一块存储区域，在实际部署过程中发现PersistentVolumeClaim 要小于 PersistentVolume。
    而PersistentVolumeClaim主要是用来提供给Elasticsearch将容器内部文件挂载到实际的服务器的硬盘中，这和用docker部署NGINX曾经做的是类似的。
## 2、Kibana需要连接Elasticsearch。通常会有一个Kibana server not ready 这样一个错误，就可以考虑这里配置的是不是有问题了。
    在kibana.yml中有这样一段配置
    ```bash
    env:
       - name: "ELASTICSEARCH_URL"
         value: "http://elasticsearch-master:9200"
    ```
    这个配置就是指向Elasticsearch，那么http://elasticsearch-master:9200这个地址是怎么来的，9200就是Elasticsearch的端口，elasticsearch-master是通过es-master.yml启动的服务的名字
    在k8s可以直接用这个访问，目前对这个我也不是很了解，目前是知道这样可以访问。
## 3、Elasticsearch配置主节点
    如果上一步，配置好了后还是有第二条提到的错误，可以查看Kibana的日志，可能会看到这样的提示
    ```
    Elasticsearch cluster did not respond with license information.
    ```
    
    这个问题，通过我查询到的原因就是没有指定主要节点。
    在es-master.yml中已经设置了这个属性，应该是不会出现这个问题了。
    ```
     - name: cluster.initial_master_nodes
       value: "elasticsearch-master-0"
    ```
    value就是启动的Elasticsearch主节点的名称
## 4、其他
### 4、1
上面有提到查看Kibana的日志，可以用下面的命令
```bash
kubectl logs 启动的Kibana的名字 -n ns-elastic
```
启动的Kibana的名字可以用下面的命令查看
```shell script
kubectl get pods -n ns-elastic # -n ns-elastic是指定namespace，ns-elastic就是之前声明的namespace
```

### 4、2
上面还有提到Elasticsearch节点的名称，可以使用如下命令
```shell script
curl http://localhost:29200/_nodes/ # 29200是本次部署中向外暴露的端口，指的是外网可以访问的端口
```
如果要输密码，密码是：changeme