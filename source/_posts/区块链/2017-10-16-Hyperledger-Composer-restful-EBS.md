---
title: Hyperledger Composer REST Server为传统ERP系统打开区块链大门
date: 2017-10-16 00:00:00
author: robin zheng
cover: true
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171016Capture2.PNG
categories: 区块链
tags:
  - Hyperledger
  - Blockchain
  - Fabric
  - Supplychain
  - RESTServer
---
本文将带您了解Hyperledger Composer REST Server 如何为传统应用系统打开区块链大门，通过使用开放的service API，可以轻松与现有系统集成在一起。通过集成现有系统将允许您从现有的业务系统中提取数据，并将其转换为一个区块链业务网络中的资产或参与者。

### 部署Hyperledger Composer REST Server

首先从github获得演示代码

```bash
root@ubuntu:~# git clone https://github.com/zhengr/hyperledger-composer-sample-applications.git
Cloning into 'hyperledger-composer-sample-applications'...
remote: Counting objects: 369, done.
remote: Compressing objects: 100% (300/300), done.
remote: Total 369 (delta 53), reused 365 (delta 51), pack-reused 0
Receiving objects: 100% (369/369), 30.86 MiB | 3.67 MiB/s, done.
Resolving deltas: 100% (53/53), done.
Checking connectivity... done.
Checking out files: 100% (304/304), done.
root@ubuntu:~#
```

然后进入创建的目录hyperledger-composer-sample-applications

```shell
root@ubuntu:~# cd hyperledger-composer-sample-applications
root@ubuntu:~/hyperledger-composer-sample-applications# ls
getTools.sh  info.js  lerna.json  LICENSE  package.json  packages  README.md  scripts
root@ubuntu:~/hyperledger-composer-sample-applications#
```

执行以下操作

```
npm install
node info.js
```

上述操作成功结束后，然后切换工作目录

```shell
root@ubuntu:~/hyperledger-composer-sample-applications# ls
getTools.sh  info.js  lerna.json  LICENSE  package.json  packages  README.md  scripts
root@ubuntu:~/hyperledger-composer-sample-applications# cd packages
root@ubuntu:~/hyperledger-composer-sample-applications/packages# ls
digitalproperty-app  vehicle-lifecycle  vehicle-lifecycle-car-builder  vehicle-lifecycle-manufacturing  vehicle-lifecycle-vda
root@ubuntu:~/hyperledger-composer-sample-applications/packages# cd vehicle-lifecycle
root@ubuntu:~/hyperledger-composer-sample-applications/packages/vehicle-lifecycle# ls
build.sh  installers  package.json  README.md
root@ubuntu:~/hyperledger-composer-sample-applications/packages/vehicle-lifecycle#
```

然后在 `vehicle-lifecycle` 目录里 执行 `./build.sh` ，这个命令将创建一个 `install.sh` 脚本文件（在目录installers/hlfv1 下）

在 `vehicle-lifecycle` 目录里继续执行 `cat installers/hlfv1/install.sh | bash` ，这个Shell脚本将自动部署数个Docker的容器。

> 注意：这个脚本将删除已经运行的其他Docker容器。

```shell
CONTAINER ID        IMAGE                                                                                                                COMMAND                  CREATED             STATUS              PORTS                                            NAMES
9310728fb3b0        nodered/node-red-docker                                                                                              "npm start -- --us..."   2 days ago          Up 2 days           0.0.0.0:1880->1880/tcp                           node-red
eebdba5351f8        hyperledger/composer-rest-server:latest                                                                              "pm2-docker compos..."   2 days ago          Up 2 days           0.0.0.0:3000->3000/tcp                           rest
442c864462fe        dev-peer0.org1.example.com-org-acme-biznet-0.13.2-5987477a7f4960eb02b7387b832090d16530cadff98bad07039443836906c55f   "chaincode -peer.a..."   2 days ago          Up 2 days                                                            dev-peer0.org1.example.com-org-acme-biznet-0.13.2
ad9b38616b3b        hyperledger/composer-playground:latest                                                                               "pm2-docker compos..."   2 days ago          Up 2 days           0.0.0.0:8080->8080/tcp                           composer
1dfce8981c7e        hyperledger/fabric-peer:x86_64-1.0.1                                                                                 "peer node start -..."   2 days ago          Up 2 days           0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
2d3e1e6a4b4b        hyperledger/fabric-orderer:x86_64-1.0.1                                                                              "orderer"                2 days ago          Up 2 days           0.0.0.0:7050->7050/tcp                           orderer.example.com
1d681fd8ce48        hyperledger/fabric-ca:x86_64-1.0.1                                                                                   "sh -c 'fabric-ca-..."   2 days ago          Up 2 days           0.0.0.0:7054->7054/tcp                           ca.org1.example.com
0dbd4d06783a        hyperledger/fabric-couchdb:x86_64-1.0.1                                                                              "tini -- /docker-e..."   2 days ago          Up 2 days           4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
root@ubuntu:~#
```

![20171016Capture1](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171016Capture1.PNG)

至此， Hyperledger Fabric， Hyperledger Composer & REST server 都已正常运行。

### 测试REST server 调用

![20171016Capture2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171016Capture2.PNG)

通过调用车辆资产的POST API将创建一个车辆资产。测试Json数据如下：

```json
{
  "$class": "org.vda.Vehicle",
  "vin": "rz-vin:3066",
  "vehicleDetails": {
    "$class": "org.vda.VehicleDetails",
    "make": "Eu enim minim.",
    "modelType": "Veniam.",
    "colour": "Sint tempor."
  },
  "vehicleStatus": "ACTIVE"
}
```

反馈结果如下：

#### Curl

```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{ \ 
   "$class": "org.vda.Vehicle", \ 
   "vin": "rz-vin:3066", \ 
   "vehicleDetails": { \ 
     "$class": "org.vda.VehicleDetails", \ 
     "make": "Eu enim minim.", \ 
     "modelType": "Veniam.", \ 
     "colour": "Sint tempor." \ 
   }, \ 
   "vehicleStatus": "ACTIVE" \ 
 }' 'http://192.168.1.185:3000/api/Vehicle'
```

#### Request URL

```
http://192.168.1.185:3000/api/Vehicle
```

#### Response Body

```
{
  "$class": "org.vda.Vehicle",
  "vin": "rz-vin:3066",
  "vehicleDetails": {
    "$class": "org.vda.VehicleDetails",
    "make": "Eu enim minim.",
    "modelType": "Veniam.",
    "colour": "Sint tempor."
  },
  "vehicleStatus": "ACTIVE"
}
```

#### Response Code

```
200
```

#### Response Headers

```
{
  "date": "Tue, 17 Oct 2017 18:06:23 GMT",
  "x-content-type-options": "nosniff",
  "etag": "W/\"c4-c8BOihCzY1zK45OdZsPEX8+ZWX4\"",
  "x-download-options": "noopen",
  "x-frame-options": "DENY",
  "content-type": "application/json; charset=utf-8",
  "access-control-allow-origin": "http://192.168.1.185:3000",
  "access-control-allow-credentials": "true",
  "connection": "keep-alive",
  "vary": "Origin, Accept-Encoding",
  "content-length": "196",
  "x-xss-protection": "1; mode=block"
}
```

反馈调用成功。

再通过调用Vehicle的GET /Vehicle/{id} 来查询，可以看到该资产已经成功添加。

![20171016Capture3](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171016Capture3.PNG)

#### Curl

```
curl -X GET --header 'Accept: application/json' 'http://192.168.1.185:3000/api/Vehicle/rz-vin%3A3066'
```

#### Request URL

```
http://192.168.1.185:3000/api/Vehicle/rz-vin%3A3066
```

#### Response Body

```
{
  "$class": "org.vda.Vehicle",
  "vin": "rz-vin:3066",
  "vehicleDetails": {
    "$class": "org.vda.VehicleDetails",
    "make": "Eu enim minim.",
    "modelType": "Veniam.",
    "colour": "Sint tempor."
  },
  "vehicleStatus": "ACTIVE"
}
```

#### Response Code

```
200
```

#### Response Headers

```
{
  "date": "Wed, 18 Oct 2017 19:08:24 GMT",
  "x-content-type-options": "nosniff",
  "etag": "W/\"c4-c8BOihCzY1zK45OdZsPEX8+ZWX4\"",
  "x-download-options": "noopen",
  "x-frame-options": "DENY",
  "content-type": "application/json; charset=utf-8",
  "access-control-allow-credentials": "true",
  "connection": "keep-alive",
  "vary": "Origin, Accept-Encoding",
  "content-length": "196",
  "x-xss-protection": "1; mode=block"
}
```

### Oracle EBS 集成思路

假设实施Oracle EBS R12的公司是一个车辆制造企业，通过订阅Oracle EBS workflow的business event中的在制品完工事件 （racle.apps.wip.job.assembly.complete）， 

![20171016Capture4](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171016Capture4.PNG)

当一辆新车在WIP中完工下线， 系统将会自动触发这个订阅， 推送编排好的数据消息到Hyperledger Composer 的REST server， 从而在Hyperledger Fabric区块链上新增了这个车辆资产。

同理， 如果Oracle EBS的SO模块完成了发运确认Ship Confirm，也通过相应的business event，这个资产就会在 Hyperledger Fabric 区块链上完成资产所有权的转移，所有的这些业务变化，都是在各个企业的共同参与的一个大的区块链上完成。

> 这样一来， 传统的ERP系统通过区块链极大的实现了业务延展，ERP将不再是某个公司单一的内部业务系统，通过区块链，各个成员公司的SAP， Oracle， JDE， Sage等传统ERP将融合智能合约共同编排成一个更大的业务数据网。基于区块链技术，任何基于数字交易的活动成本和追踪成本都会降低，并且能提高安全性。
>
> 随着技术的进一步成熟，区块链势必将为企业的诚信经营保驾护航。

#### Ref

* [如何使用Oracle EBS的Business Event](http://awads.net/wp/2011/07/21/how-to-create-and-use-custom-business-events-in-oracle-e-business-suite/) - How To Create and Use Custom Business Events in Oracle E-Business Suite
* [Oracle EBS Business Event官方文档](https://docs.oracle.com/cd/E18727_01/doc.121/e12905/T361836T361840.htm) - Managing Business Events 
