---
title: 详解区块链Hyperledger Fabric资产（车辆）交易
date: 2017-05-29 00:00:00
author: robin zheng
cover: true
img: https://hyperledger-fabric.readthedocs.io/en/latest/_images/hyperledger_fabric_logo_color.png
categories: 区块链
tags:
  - Hyperledger
  - Blockchain
  - Supplychain
---

## 详解区块链Hyperledger Fabric资产（车辆）交易

首先定义Chaincode 链码及API， 通过定义相应的数据结构和方法就完成了一个智能契约，参考原文如下：

> Chaincode is a program, written in Go, and eventually in other programming languages such as Java, that implements a prescribed interface. Chaincode runs in a secured Docker container isolated from the endorsing peer process. Chaincode initializes and manages ledger state through transactions submitted by applications. A chaincode typically handles business logic agreed to by members of the network, so it may be considered as a “smart contract”. State created by a chaincode is scoped exclusively to that chaincode and can’t be accessed directly by another chaincode. However, within the same network, given the appropriate permission a chaincode may invoke another chaincode to access its state.
>

下面fabcar这个chaincode里car有四属性

- Make   string `json:"make"`
- Model  string `json:"model"`
- Colour string `json:"colour"`
- Owner  string `json:"owner"`

以及各类交易方法:

1.  createCar   创建车
2.  queryAllCars 查询所有车
3.  changeCarOwner  改变车主

chaincode具体代码如下：

```go
/*
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied.  See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */

/*
 * The sample smart contract for documentation topic:
 * Writing Your First Blockchain Application
 */

package main

/* Imports
 * 4 utility libraries for formatting, handling bytes, reading and writing JSON, and string manipulation
 * 2 specific Hyperledger Fabric specific libraries for Smart Contracts
 */
import (
	"bytes"
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	sc "github.com/hyperledger/fabric/protos/peer"
)

// Define the Smart Contract structure
type SmartContract struct {
}

// Define the car structure, with 4 properties.  Structure tags are used by encoding/json library 建立数据结构
type Car struct {
	Make   string `json:"make"`
	Model  string `json:"model"`
	Colour string `json:"colour"`
	Owner  string `json:"owner"`
}

/*
 * The Init method is called when the Smart Contract "fabcar" is instantiated by the blockchain network
 * Best practice is to have any Ledger initialization in separate function -- see initLedger()
 */
func (s *SmartContract) Init(APIstub shim.ChaincodeStubInterface) sc.Response {
	return shim.Success(nil)
}

/*
 * The Invoke method is called as a result of an application request to run the Smart Contract "fabcar"
 * The calling application program has also specified the particular smart contract function to be called, with arguments
 */
func (s *SmartContract) Invoke(APIstub shim.ChaincodeStubInterface) sc.Response {

	// Retrieve the requested Smart Contract function and arguments
	function, args := APIstub.GetFunctionAndParameters()
	// Route to the appropriate handler function to interact with the ledger appropriately
	if function == "queryCar" {
		return s.queryCar(APIstub, args)
	} else if function == "initLedger" {
		return s.initLedger(APIstub)
	} else if function == "createCar" {
		return s.createCar(APIstub, args)
	} else if function == "queryAllCars" {
		return s.queryAllCars(APIstub)
	} else if function == "changeCarOwner" {
		return s.changeCarOwner(APIstub, args)
	}

	return shim.Error("Invalid Smart Contract function name.")
}

func (s *SmartContract) queryCar(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting 1")
	}

	carAsBytes, _ := APIstub.GetState(args[0])
	return shim.Success(carAsBytes)
}
// 初始化账簿10辆车
func (s *SmartContract) initLedger(APIstub shim.ChaincodeStubInterface) sc.Response {
	cars := []Car{
		Car{Make: "Toyota", Model: "Prius", Colour: "blue", Owner: "Tomoko"},
		Car{Make: "Ford", Model: "Mustang", Colour: "red", Owner: "Brad"},
		Car{Make: "Hyundai", Model: "Tucson", Colour: "green", Owner: "Jin Soo"},
		Car{Make: "Volkswagen", Model: "Passat", Colour: "yellow", Owner: "Max"},
		Car{Make: "Tesla", Model: "S", Colour: "black", Owner: "Adriana"},
		Car{Make: "Peugeot", Model: "205", Colour: "purple", Owner: "Michel"},
		Car{Make: "Chery", Model: "S22L", Colour: "white", Owner: "Aarav"},
		Car{Make: "Fiat", Model: "Punto", Colour: "violet", Owner: "Pari"},
		Car{Make: "Tata", Model: "Nano", Colour: "indigo", Owner: "Valeria"},
		Car{Make: "Holden", Model: "Barina", Colour: "brown", Owner: "Shotaro"},
	}

	i := 0
	for i < len(cars) {
		fmt.Println("i is ", i)
		carAsBytes, _ := json.Marshal(cars[i])
		APIstub.PutState("CAR"+strconv.Itoa(i), carAsBytes)
		fmt.Println("Added", cars[i])
		i = i + 1
	}

	return shim.Success(nil)
}

func (s *SmartContract) createCar(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	if len(args) != 5 {
		return shim.Error("Incorrect number of arguments. Expecting 5")
	}

	var car = Car{Make: args[1], Model: args[2], Colour: args[3], Owner: args[4]}

	carAsBytes, _ := json.Marshal(car)
	APIstub.PutState(args[0], carAsBytes)

	return shim.Success(nil)
}

func (s *SmartContract) queryAllCars(APIstub shim.ChaincodeStubInterface) sc.Response {

	startKey := "CAR0"
	endKey := "CAR999"

	resultsIterator, err := APIstub.GetStateByRange(startKey, endKey)
	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryResults
	var buffer bytes.Buffer
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}
		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}
		buffer.WriteString("{\"Key\":")
		buffer.WriteString("\"")
		buffer.WriteString(queryResponse.Key)
		buffer.WriteString("\"")

		buffer.WriteString(", \"Record\":")
		// Record is a JSON object, so we write as-is
		buffer.WriteString(string(queryResponse.Value))
		buffer.WriteString("}")
		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	fmt.Printf("- queryAllCars:\n%s\n", buffer.String())

	return shim.Success(buffer.Bytes())
}

func (s *SmartContract) changeCarOwner(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	carAsBytes, _ := APIstub.GetState(args[0])
	car := Car{}

	json.Unmarshal(carAsBytes, &car)
	car.Owner = args[1]

	carAsBytes, _ = json.Marshal(car)
	APIstub.PutState(args[0], carAsBytes)

	return shim.Success(nil)
}

// The main function is only relevant in unit test mode. Only included here for completeness.
func main() {

	// Create a new Smart Contract
	err := shim.Start(new(SmartContract))
	if err != nil {
		fmt.Printf("Error creating new Smart Contract: %s", err)
	}
}
```

### 构建区块链网络

启动脚本：startFabric.sh

```
#!/bin/bash
# Exit on first error
set -e

starttime=$(date +%s)

if [ ! -d ~/.hfc-key-store/ ]; then
	mkdir ~/.hfc-key-store/
fi
cp $PWD/creds/* ~/.hfc-key-store/
# launch network; create channel and join peer to channel  
# 启动区域连网络，并把各peer 加入Chanel
cd ../basic-network
./start.sh

# Now launch the CLI container in order to install, instantiate chaincode 
# 启动Cli容器并安装，初始化链码
# and prime the ledger with our 10 cars
# 然后初始化10辆车
docker-compose -f ./docker-compose.yml up -d cli

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode install -n fabcar -v 1.0 -p github.com/fabcar
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n fabcar -v 1.0 -c '{"Args":[""]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
sleep 10
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"initLedger","Args":[""]}'

printf "\nTotal execution time : $(($(date +%s) - starttime)) secs ...\n\n"
```

执行脚本


```shell
root@ubuntu:~/fabcar# . ./startFabric.sh

docker-compose -f docker-compose.yml down
Removing network net_basic

docker-compose -f docker-compose.yml up -d ca.example.com orderer.example.com peer0.org1.example.com couchdb
Creating network "net_basic" with the default driver
Creating couchdb ... 
Creating ca.example.com ... 
Creating orderer.example.com ... 
Creating ca.example.com
Creating couchdb
Creating orderer.example.com ... done
Creating peer0.org1.example.com ... 
Creating peer0.org1.example.com ... done

# wait for Hyperledger Fabric to start
# incase of errors when running later commands, issue export FABRIC_START_TIMEOUT=<larger number>
export FABRIC_START_TIMEOUT=10
#echo ${FABRIC_START_TIMEOUT}
sleep ${FABRIC_START_TIMEOUT}

# Create the channel
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
2017-08-29 19:12:19.929 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-08-29 19:12:19.929 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-08-29 19:12:19.933 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2017-08-29 19:12:19.933 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-08-29 19:12:19.933 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-08-29 19:12:19.935 UTC [msp] GetLocalMSP -> DEBU 006 Returning existing local MSP
2017-08-29 19:12:19.935 UTC [msp] GetDefaultSigningIdentity -> DEBU 007 Obtaining default signing identity
2017-08-29 19:12:19.935 UTC [msp/identity] Sign -> DEBU 008 Sign: plaintext: 0A8C060A074F7267314D53501280062D...53616D706C65436F6E736F727469756D 
2017-08-29 19:12:19.935 UTC [msp/identity] Sign -> DEBU 009 Sign: digest: AB4CF53AE7F25F55FC853E7AA8FA847F9ADB7F5669C251C029313639CBE3EB0C 
2017-08-29 19:12:19.935 UTC [msp] GetLocalMSP -> DEBU 00a Returning existing local MSP
2017-08-29 19:12:19.935 UTC [msp] GetDefaultSigningIdentity -> DEBU 00b Obtaining default signing identity
2017-08-29 19:12:19.936 UTC [msp] GetLocalMSP -> DEBU 00c Returning existing local MSP
2017-08-29 19:12:19.936 UTC [msp] GetDefaultSigningIdentity -> DEBU 00d Obtaining default signing identity
2017-08-29 19:12:19.936 UTC [msp/identity] Sign -> DEBU 00e Sign: plaintext: 0AC3060A1508021A060893F996CD0522...04CD8921E4BA83A1D6A2088DF251B740 
2017-08-29 19:12:19.936 UTC [msp/identity] Sign -> DEBU 00f Sign: digest: BF6CC7A8635CE11B92CE895142A4377D9A70CB14EA9ACBF53CD39E5E1E010F06 
2017-08-29 19:12:20.084 UTC [msp] GetLocalMSP -> DEBU 010 Returning existing local MSP
2017-08-29 19:12:20.084 UTC [msp] GetDefaultSigningIdentity -> DEBU 011 Obtaining default signing identity
2017-08-29 19:12:20.084 UTC [msp] GetLocalMSP -> DEBU 012 Returning existing local MSP
2017-08-29 19:12:20.084 UTC [msp] GetDefaultSigningIdentity -> DEBU 013 Obtaining default signing identity
2017-08-29 19:12:20.084 UTC [msp/identity] Sign -> DEBU 014 Sign: plaintext: 0AC3060A1508021A060894F996CD0522...9BCAC7F7286212080A021A0012021A00 
2017-08-29 19:12:20.084 UTC [msp/identity] Sign -> DEBU 015 Sign: digest: 3FD3A5C2FA53214CEFC3B47DFC4DEE78DF2BBC1646C77C2CA3039A7E83378931 
2017-08-29 19:12:20.086 UTC [channelCmd] readBlock -> DEBU 016 Got status:*orderer.DeliverResponse_Status 
2017-08-29 19:12:20.086 UTC [msp] GetLocalMSP -> DEBU 017 Returning existing local MSP
2017-08-29 19:12:20.086 UTC [msp] GetDefaultSigningIdentity -> DEBU 018 Obtaining default signing identity
2017-08-29 19:12:20.090 UTC [channelCmd] InitCmdFactory -> INFO 019 Endorser and orderer connections initialized
2017-08-29 19:12:20.290 UTC [msp] GetLocalMSP -> DEBU 01a Returning existing local MSP
2017-08-29 19:12:20.290 UTC [msp] GetDefaultSigningIdentity -> DEBU 01b Obtaining default signing identity
2017-08-29 19:12:20.290 UTC [msp] GetLocalMSP -> DEBU 01c Returning existing local MSP
2017-08-29 19:12:20.290 UTC [msp] GetDefaultSigningIdentity -> DEBU 01d Obtaining default signing identity
2017-08-29 19:12:20.290 UTC [msp/identity] Sign -> DEBU 01e Sign: plaintext: 0AC3060A1508021A060894F996CD0522...AA1D768A0B9D12080A021A0012021A00 
2017-08-29 19:12:20.290 UTC [msp/identity] Sign -> DEBU 01f Sign: digest: 560CFC8CB4E9D591BE81CBD50A50107BE8B074021595CBB1F9DFD8EC5A78C857 
2017-08-29 19:12:20.297 UTC [channelCmd] readBlock -> DEBU 020 Received block:0 
2017-08-29 19:12:20.299 UTC [main] main -> INFO 021 Exiting.....
# Join peer0.org1.example.com to the channel.
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block
2017-08-29 19:12:20.602 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-08-29 19:12:20.602 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-08-29 19:12:20.607 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2017-08-29 19:12:20.610 UTC [msp/identity] Sign -> DEBU 004 Sign: plaintext: 0A8A070A5C08011A0C0894F996CD0510...9BD6C0AB03541A080A000A000A000A00 
2017-08-29 19:12:20.610 UTC [msp/identity] Sign -> DEBU 005 Sign: digest: F428B9E87CAC4F0EC5EB57736BDCCBAE1E1972C69B77467D2EE41B783867FCA3 
2017-08-29 19:12:20.813 UTC [channelCmd] executeJoin -> INFO 006 Peer joined the channel!
2017-08-29 19:12:20.813 UTC [main] main -> INFO 007 Exiting.....
Pulling cli (hyperledger/fabric-tools:x86_64-1.0.0)...
x86_64-1.0.0: Pulling from hyperledger/fabric-tools
aafe6b5e13de: Already exists
0a2b43a72660: Already exists
18bdd1e546d2: Already exists
8198342c3e05: Already exists
f56970a44fd4: Already exists
e32b597e7839: Already exists
a6e362fc71c4: Already exists
e21570e54f61: Already exists
66620db3cda6: Already exists
0548c619fbb7: Already exists
e084aff94b0b: Already exists
fac34f94b9b9: Pull complete
d4ca6b61a36e: Pull complete
6378740a852a: Pull complete
75a35cd33119: Pull complete
Digest: sha256:c107430c14344f4f37f0882f3eb8591520abd699a0b9da2b507f7527505612a7
Status: Downloaded newer image for hyperledger/fabric-tools:x86_64-1.0.0
Creating cli ... 
Creating cli ... done
2017-08-29 19:12:38.644 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-08-29 19:12:38.645 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-08-29 19:12:38.645 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-08-29 19:12:38.645 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-08-29 19:12:39.397 UTC [golang-platform] getCodeFromFS -> DEBU 005 getCodeFromFS github.com/fabcar
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 006 Discarding GOROOT package bytes
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 007 Discarding GOROOT package encoding/json
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 008 Discarding GOROOT package fmt
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 009 Discarding provided package github.com/hyperledger/fabric/core/chaincode/shim
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 00a Discarding provided package github.com/hyperledger/fabric/protos/peer
2017-08-29 19:12:40.404 UTC [golang-platform] func1 -> DEBU 00b Discarding GOROOT package strconv
2017-08-29 19:12:40.405 UTC [golang-platform] GetDeploymentPayload -> DEBU 00c done
2017-08-29 19:12:40.412 UTC [msp/identity] Sign -> DEBU 00d Sign: plaintext: 0A8A070A5C08031A0C08A8F996CD0510...939FFF060000FFFF9C08DC0700200000 
2017-08-29 19:12:40.412 UTC [msp/identity] Sign -> DEBU 00e Sign: digest: 46FFD3912E51469B57E87B5832DF2FC27CCABA13316494E0418B2B600C2233DA 
2017-08-29 19:12:40.419 UTC [chaincodeCmd] install -> DEBU 00f Installed remotely response:<status:200 payload:"OK" > 
2017-08-29 19:12:40.419 UTC [main] main -> INFO 010 Exiting.....
2017-08-29 19:12:40.631 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-08-29 19:12:40.631 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-08-29 19:12:40.632 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-08-29 19:12:40.632 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-08-29 19:12:40.634 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0A95070A6708031A0C08A8F996CD0510...324D53500A04657363630A0476736363 
2017-08-29 19:12:40.634 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: 85C3D93AD44FDAAF2A1A92BC1FC4DAEFD2E1F796880B36A86673E4E3CE266EBD 
2017-08-29 19:13:53.027 UTC [msp/identity] Sign -> DEBU 007 Sign: plaintext: 0A95070A6708031A0C08A8F996CD0510...57E2BBF703C567C9EC8E912FECD8AE3D 
2017-08-29 19:13:53.027 UTC [msp/identity] Sign -> DEBU 008 Sign: digest: 7132F6CEBF94F7C478AC63DFE841FDBEE31EC4C78B67283AB373A2D811A5BEE1 
2017-08-29 19:13:53.032 UTC [main] main -> INFO 009 Exiting.....
2017-08-29 19:14:03.355 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-08-29 19:14:03.355 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-08-29 19:14:03.358 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-08-29 19:14:03.358 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-08-29 19:14:03.360 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0A97070A6908031A0C08FBF996CD0510...1A0E0A0A696E69744C65646765720A00 
2017-08-29 19:14:03.360 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: EBA72A2A263CFB4245DB7C51BCD134E981CB0229F5022E8C485FBDAF7B3B703B 
2017-08-29 19:14:03.412 UTC [msp/identity] Sign -> DEBU 007 Sign: plaintext: 0A97070A6908031A0C08FBF996CD0510...D9166A19C3A967F8BEE99CABD206C9D6 
2017-08-29 19:14:03.412 UTC [msp/identity] Sign -> DEBU 008 Sign: digest: D5386F73796B0FCE332877D1ACF454DDE073899A609867B68B35F459382A2F66 
2017-08-29 19:14:03.418 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 009 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \301G\277h\207t\276[\r\314\30631um\345J[#\356%0\010\320\317\n\016y\255\356\322\322\022\267\006\n\240\006\022\205\006\n\006fabcar\022\372\005\032J\n\004CAR0\032B{\"make\":\"Toyota\",\"model\":\"Prius\",\"colour\":\"blue\",\"owner\":\"Tomoko\"}\032G\n\004CAR1\032?{\"make\":\"Ford\",\"model\":\"Mustang\",\"colour\":\"red\",\"owner\":\"Brad\"}\032N\n\004CAR2\032F{\"make\":\"Hyundai\",\"model\":\"Tucson\",\"colour\":\"green\",\"owner\":\"Jin Soo\"}\032N\n\004CAR3\032F{\"make\":\"Volkswagen\",\"model\":\"Passat\",\"colour\":\"yellow\",\"owner\":\"Max\"}\032G\n\004CAR4\032?{\"make\":\"Tesla\",\"model\":\"S\",\"colour\":\"black\",\"owner\":\"Adriana\"}\032K\n\004CAR5\032C{\"make\":\"Peugeot\",\"model\":\"205\",\"colour\":\"purple\",\"owner\":\"Michel\"}\032H\n\004CAR6\032@{\"make\":\"Chery\",\"model\":\"S22L\",\"colour\":\"white\",\"owner\":\"Aarav\"}\032H\n\004CAR7\032@{\"make\":\"Fiat\",\"model\":\"Punto\",\"colour\":\"violet\",\"owner\":\"Pari\"}\032J\n\004CAR8\032B{\"make\":\"Tata\",\"model\":\"Nano\",\"colour\":\"indigo\",\"owner\":\"Valeria\"}\032M\n\004CAR9\032E{\"make\":\"Holden\",\"model\":\"Barina\",\"colour\":\"brown\",\"owner\":\"Shotaro\"}\022\026\n\004lscc\022\016\n\014\n\006fabcar\022\002\010\001\032\003\010\310\001\"\r\022\006fabcar\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\374\005-----BEGIN -----\nMIICGDCCAb+gAwIBAgIQPcMFFEB/vq6mEL6vXV7aUTAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0xNzA2MjMxMjMzMTlaFw0yNzA2MjExMjMzMTla\nMFsxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMR8wHQYDVQQDExZwZWVyMC5vcmcxLmV4YW1wbGUuY29tMFkw\nEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEzS9k2gCKHcat8Wj4T2nB1uyC8R2zg3um\nxdTL7nmgFWp0uyCCbQQxD/VS+8R/3DNvEFkvzhcjc9NU/nRqMirpLqNNMEswDgYD\nVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0jBCQwIoAgDnKSJOiz8xeE\nyKk8W4729MHJHZ5uV3xFwzFjYJ/kABEwCgYIKoZIzj0EAwIDRwAwRAIgHBdxbHUG\nrFUzKPX9UmmN3SwigWcRUREUy/GTb3hDIAsCIEF1BxTqv8ilQYE8ql0wJL4mTber\nHE6DFYvvBCUnicUh\n-----END -----\n" signature:"0E\002!\000\234s\024\311\257\272\315\375:\355z\260}\336X\214\034\372MP\374L`\251\0034\217\327g\274\300\263\002 \rD]\340\332\343\032+\312\364vox\277JV\331\026j\031\303\251g\370\276\351\234\253\322\006\311\326" > 
2017-08-29 19:14:03.419 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00a Chaincode invoke successful. result: status:200 
2017-08-29 19:14:03.419 UTC [main] main -> INFO 00b Exiting.....

Total execution time : 122 secs ...

root@ubuntu:~/basic-network#
```

```shell
CONTAINER ID        IMAGE                                     COMMAND                  CREATED             STATUS              PORTS                                            NAMES
3b2978bd8bb9        dev-peer0.org1.example.com-fabcar-1.0     "chaincode -peer.a..."   15 hours ago        Up 15 hours                                                          dev-peer0.org1.example.com-fabcar-1.0
e58f60336983        hyperledger/fabric-tools:x86_64-1.0.0     "/bin/bash"              15 hours ago        Up 15 hours                                                          cli
efecad2b6d36        hyperledger/fabric-peer:x86_64-1.0.0      "peer node start"        15 hours ago        Up 15 hours         0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
8bf3d70e5f42        hyperledger/fabric-ca:x86_64-1.0.0        "sh -c 'fabric-ca-..."   15 hours ago        Up 15 hours         0.0.0.0:7054->7054/tcp                           ca.example.com
dd58d8c032e3        hyperledger/fabric-orderer:x86_64-1.0.0   "orderer"                15 hours ago        Up 15 hours         0.0.0.0:7050->7050/tcp                           orderer.example.com
8077c0b69096        hyperledger/fabric-couchdb:x86_64-1.0.0   "tini -- /docker-e..."   15 hours ago        Up 15 hours         4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
```

### 执行区块链交易

通过以上过程完成构建一个Blockchain Network，接下来我们将通过Node.Js编写的程序Application执行交易。

![fabcar](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/fabcar.png)

![fabcar2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/fabcar2.png)

安装Fabric Node SDK

```shell
root@ubuntu:~/fabcar# npm install
```

编写查询程序query.js

```javascript
'use strict';
/*
*
* SPDX-License-Identifier: Apache-2.0
*/
/*
 * Hyperledger Fabric Sample Query Program
 */

var hfc = require('fabric-client');
var path = require('path');

var options = {
    wallet_path: path.join(__dirname, './creds'),
    user_id: 'PeerAdmin',
    channel_id: 'mychannel',
    chaincode_id: 'fabcar',
    network_url: 'grpc://localhost:7051',
};

var channel = {};
var client = null;

Promise.resolve().then(() => {
    console.log("Create a client and set the wallet location");
    client = new hfc();
    return hfc.newDefaultKeyValueStore({ path: options.wallet_path });
}).then((wallet) => {
    console.log("Set wallet path, and associate user ", options.user_id, " with application");
    client.setStateStore(wallet);
    return client.getUserContext(options.user_id, true);
}).then((user) => {
    console.log("Check user is enrolled, and set a query URL in the network");
    if (user === undefined || user.isEnrolled() === false) {
        console.error("User not defined, or not enrolled - error");
    }
    channel = client.newChannel(options.channel_id);
    channel.addPeer(client.newPeer(options.network_url));
    return;
}).then(() => {
    console.log("Make query");
    var transaction_id = client.newTransactionID();
    console.log("Assigning transaction_id: ", transaction_id._transaction_id);

    // queryCar - requires 1 argument, ex: args: ['CAR4'],
    // queryAllCars - requires no arguments , ex: args: [''],
    const request = {
        chaincodeId: options.chaincode_id,
        txId: transaction_id,
        fcn: 'queryAllCars',
        args: ['']
    };
    return channel.queryByChaincode(request);
}).then((query_responses) => {
    console.log("returned from query");
    if (!query_responses.length) {
        console.log("No payloads were returned from query");
    } else {
        console.log("Query result count = ", query_responses.length)
    }
    if (query_responses[0] instanceof Error) {
        console.error("error from query = ", query_responses[0]);
    }
    console.log("Response is ", query_responses[0].toString());
}).catch((err) => {
    console.error("Caught Error", err);
});
```

执行query.js

```
root@ubuntu:~/fabcar# node query.js
Create a client and set the wallet location
Set wallet path, and associate user  PeerAdmin  with application
Check user is enrolled, and set a query URL in the network
Make query
Assigning transaction_id:  aa4384c6e00c25965bfd7690468003ae5c602e3d70df5eccca6dbf9657cce077
returned from query
Query result count =  1
Response is  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
root@ubuntu:~/fabcar# 

```

改变车主， 把"Adriana"的"CAR4", "Tesla","model":"S" 的owner改为"Zheng rui"

changeowner.js

```javascript
'use strict';
/*
* Copyright IBM Corp All Rights Reserved
*
* SPDX-License-Identifier: Apache-2.0
*/
/*
 * Chaincode Invoke
 */

var hfc = require('fabric-client');
var path = require('path');
var util = require('util');

var options = {
    wallet_path: path.join(__dirname, './creds'),
    user_id: 'PeerAdmin',
    channel_id: 'mychannel',
    chaincode_id: 'fabcar',
    peer_url: 'grpc://localhost:7051',
    event_url: 'grpc://localhost:7053',
    orderer_url: 'grpc://localhost:7050'
};

var channel = {};
var client = null;
var targets = [];
var tx_id = null;
Promise.resolve().then(() => {
    console.log("Create a client and set the wallet location");
    client = new hfc();
    return hfc.newDefaultKeyValueStore({ path: options.wallet_path });
}).then((wallet) => {
    console.log("Set wallet path, and associate user ", options.user_id, " with application");
    client.setStateStore(wallet);
    return client.getUserContext(options.user_id, true);
}).then((user) => {
    console.log("Check user is enrolled, and set a query URL in the network");
    if (user === undefined || user.isEnrolled() === false) {
        console.error("User not defined, or not enrolled - error");
    }
    channel = client.newChannel(options.channel_id);
    var peerObj = client.newPeer(options.peer_url);
    channel.addPeer(peerObj);
    channel.addOrderer(client.newOrderer(options.orderer_url));
    targets.push(peerObj);
    return;
}).then(() => {
    tx_id = client.newTransactionID();
    console.log("Assigning transaction_id: ", tx_id._transaction_id);
    // createCar - requires 5 args, ex: args: ['CAR11', 'Honda', 'Accord', 'Black', 'Tom'],
    // changeCarOwner - requires 2 args , ex: args: ['CAR10', 'Barry'],
    // send proposal to endorser
    var request = {
        targets: targets,
        chaincodeId: options.chaincode_id,
        fcn: 'changeCarOwner',
        args: ['CAR4', 'Zheng rui'],
        chainId: options.channel_id,
        txId: tx_id
    };
    return channel.sendTransactionProposal(request);
}).then((results) => {
    var proposalResponses = results[0];
    var proposal = results[1];
    var header = results[2];
    let isProposalGood = false;
    if (proposalResponses && proposalResponses[0].response &&
        proposalResponses[0].response.status === 200) {
        isProposalGood = true;
        console.log('transaction proposal was good');
    } else {
        console.error('transaction proposal was bad');
    }
    if (isProposalGood) {
        console.log(util.format(
            'Successfully sent Proposal and received ProposalResponse: Status - %s, message - "%s", metadata - "%s", endorsement signature: %s',
            proposalResponses[0].response.status, proposalResponses[0].response.message,
            proposalResponses[0].response.payload, proposalResponses[0].endorsement.signature));
        var request = {
            proposalResponses: proposalResponses,
            proposal: proposal,
            header: header
        };
        // set the transaction listener and set a timeout of 30sec
        // if the transaction did not get committed within the timeout period,
        // fail the test
        var transactionID = tx_id.getTransactionID();
        var eventPromises = [];
        let eh = client.newEventHub();
        eh.setPeerAddr(options.event_url);
        eh.connect();

        let txPromise = new Promise((resolve, reject) => {
            let handle = setTimeout(() => {
                eh.disconnect();
                reject();
            }, 30000);

            eh.registerTxEvent(transactionID, (tx, code) => {
                clearTimeout(handle);
                eh.unregisterTxEvent(transactionID);
                eh.disconnect();

                if (code !== 'VALID') {
                    console.error(
                        'The transaction was invalid, code = ' + code);
                    reject();
                } else {
                    console.log(
                        'The transaction has been committed on peer ' +
                        eh._ep._endpoint.addr);
                    resolve();
                }
            });
        });
        eventPromises.push(txPromise);
        var sendPromise = channel.sendTransaction(request);
        return Promise.all([sendPromise].concat(eventPromises)).then((results) => {
            console.log(' event promise all complete and testing complete');
            return results[0]; // the first returned value is from the 'sendPromise' which is from the 'sendTransaction()' call
        }).catch((err) => {
            console.error(
                'Failed to send transaction and get notifications within the timeout period.'
            );
            return 'Failed to send transaction and get notifications within the timeout period.';
        });
    } else {
        console.error(
            'Failed to send Proposal or receive valid response. Response null or status is not 200. exiting...'
        );
        return 'Failed to send Proposal or receive valid response. Response null or status is not 200. exiting...';
    }
}, (err) => {
    console.error('Failed to send proposal due to error: ' + err.stack ? err.stack :
        err);
    return 'Failed to send proposal due to error: ' + err.stack ? err.stack :
        err;
}).then((response) => {
    if (response.status === 'SUCCESS') {
        console.log('Successfully sent transaction to the orderer.');
        return tx_id.getTransactionID();
    } else {
        console.error('Failed to order the transaction. Error code: ' + response.status);
        return 'Failed to order the transaction. Error code: ' + response.status;
    }
}, (err) => {
    console.error('Failed to send transaction due to error: ' + err.stack ? err
        .stack : err);
    return 'Failed to send transaction due to error: ' + err.stack ? err.stack :
        err;
});

```

执行改车主程序

```shell
root@ubuntu:~/fabcar# node changeowner.js
Create a client and set the wallet location
Set wallet path, and associate user  PeerAdmin  with application
Check user is enrolled, and set a query URL in the network
Assigning transaction_id:  c5263d9f94a0876ea80fcf4d3eb5ff914525afdcf521e7a21a64526b88bb023e
transaction proposal was good
Successfully sent Proposal and received ProposalResponse: Status - 200, message - "OK", metadata - "", endorsement signature: 0D 5�"���b�Dc��Hb7��-n�Hx[�� 5�{h6���g���v�� 7���DZ�6LA��
info: [EventHub.js]: _connect - options {}
The transaction has been committed on peer localhost:7053
 event promise all complete and testing complete
Successfully sent transaction to the orderer.
root@ubuntu:~/fabcar# 
```

再查询一下

```shell
root@ubuntu:~/fabcar# node query.js
Create a client and set the wallet location
Set wallet path, and associate user  PeerAdmin  with application
Check user is enrolled, and set a query URL in the network
Make query
Assigning transaction_id:  757d64fa926de8ed57c944e3e92b470d76cc37a21b9cea1cf299e7177daf7f77
returned from query
Query result count =  1
Response is  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Zheng rui"}},{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]

```

> {"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Zheng rui"}}

车主变为“Zheng rui”， 是不是很简单。

###  为什么要用区块链？

> 引一下Hyperledger的官方说法: 区块链技术将带来的广泛、根本的革命，自互联网以来，没有一个技术能与其潜力相比。区块链是一个端对端的分布式账薄，叠加了共识机制，加上“智能合约”系统和其他辅助技术。所有这些共同作用，可以用来构建新一代的交易应用，在核心上建立信任、责任和透明度，同时精简业务流程和法律约束。
>
> 我们可以把它想象为一个交易市场的操作系统、数据共享网络、微货币、以及一个去中心化的数字社区。它有极大降低现实世界中完成交易的成本和复杂性的潜能。
>
> 只有一个开源、协作的软件开发方法才能够确保所需要的透明度、长期性、互操作性和支持，将区块链技术带进主流商业应用。超级账本的宗旨是——创建从事区块链框架和平台搭建的软件开发人员社区。
