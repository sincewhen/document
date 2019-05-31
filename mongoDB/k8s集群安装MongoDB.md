# k8s集群安装MongoDB

<a name="5ecf95d5"></a>
# MongoDB安装

<a name="9990da3d"></a>
## 安装包安装

![图片.png](https://cdn.nlark.com/yuque/0/2019/png/240867/1548054456180-345d85c9-bfd8-4664-8b73-2584ba8366c5.png#align=left&display=inline&height=241&name=%E5%9B%BE%E7%89%87.png&originHeight=301&originWidth=1291&size=121216&status=done&width=1033)

配置yum源
**[mongodb-org-4.0]**
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

安装MongoDB包<br />安装最新稳定版<br />sudo yum install -y mongodb-org<br />安装特定版本，需要分别指定每个组件的版本号<br />sudo yum install -y mongodb-org-4.0.5 mongodb-org-server-4.0.5 mongodb-org-shell-4.0.5 mongodb-org-mongos-4.0.5 mongodb-org-tools-4.0.5<br />阻止yum时更新版本<br />exclude=mongodb-org,mongodb-org-server,mongodb-org-shell,mongodb-org-mongos,mongodb-org-tools
<a name="6232cdc9"></a>
## docker安装
docker run --name mongo -d mongo:4.1.6
<a name="8eba8d2b"></a>
## k8s平台安装
mongo部署文件
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: rdc-database
spec:
  type: LoadBalancer
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    app: mongodb
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mongodb
  namespace: rdc-database
spec:
  selector:
    matchLabels:
      app: mongodb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - image: hub.i139.cn/rdc/mongo:4.1.6
        name: mongodb
        ports:
        - containerPort: 27017
          name: mongodb
        volumeMounts:
        - name: mongodb-persistent-storage
          mountPath: /var/lib/mongodb
      volumes:
      - name: mongodb-persistent-storage
        persistentVolumeClaim:
          claimName: mongodb-pv-claim
```
pv部署文件

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mongodb-pv-volume
  namespace: rdc-database
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pv-claim
  namespace: rdc-database
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

<a name="5a993b66"></a>
## Shell脚本操作
插入操作

```javascript
db.inventory.insertMany([
   // MongoDB adds the _id field with an ObjectId if _id is not present
   { item: "journal", qty: 25, status: "A",
       size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "notebook", qty: 50, status: "A",
       size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
   { item: "paper", qty: 100, status: "D",
       size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
   { item: "planner", qty: 75, status: "D",
       size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "postcard", qty: 45, status: "A",
       size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
]);
```
查询操作

```javascript
#查询
db.inventory.find( {} )
#带条件查询
db.inventory.find( { status: "D" } )
#匹配内嵌集合查询
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )
#匹配内嵌集合的一个域
db.inventory.find( { "size.uom": "in" } )

```

<a name="542d6f1b"></a>
## JAVA驱动操作

导入依赖
```xml
		<dependency>
			<groupId>org.mongodb</groupId>
			<artifactId>mongodb-driver</artifactId>
			<version>3.4.3</version>
		</dependency>
```

连接数据库
```java
MongoClient client = new MongoClient("192.168.67.32",30955);
MongoDatabase databases = client.getDatabase("test");
MongoCollection<Document> collection = databases.getCollection("inventory");
```

