# Fabric 相关教程  
## 1. 使用Fabric的测试网络  
* 可以使用 *fabric-sample* 代码库中提供的脚本来部署测试网络  

注意事项：  
> 仅仅适用于教育和测试，不可用于部书产品网络的模板  
> 不要修改脚本，否则可能破坏网络  
> 基于最新的稳定版Docker镜像和提供的tar文件中的预编译的安装软件进行验证  
如使用当前的master分支的镜像或工具运行这些命令，则可能会遇到错误  

### 1.1 启动测试网络   
在 *fabric-samples* 代码库的 *test-network* 目录中找到启动网络的脚本:  
 ***cd fabric-samples/test-network***   

*network.sh* : 在本地计算机上使用Docker镜像建立Fabric网络  
***./network.sh -h*** -->  以打印脚本帮助文本   
```bash  
Usage:
  network.sh <Mode> [Flags]
    Modes:
      up - bring up fabric orderer and peer nodes. No channel is created
      up createChannel - bring up fabric network with one channel
      createChannel - create and join a channel after the network is created
      deployCC - deploy the asset transfer basic chaincode on the channel or specify
      down - clear the network with docker-compose down
      restart - restart the network

    Flags:
    -ca <use CAs> -  create Certificate Authorities to generate the crypto material
    -c <channel name> - channel name to use (defaults to "mychannel")
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - delay duration in seconds (defaults to 3)
    -ccn <name> - the short name of the chaincode to deploy: basic (default),ledger, private, secured
    -ccl <language> - the programming language of the chaincode to deploy: go (default), java, javascript, typescript
    -ccv <version>  - chaincode version. 1.0 (default)
    -ccs <sequence>  - chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
    -ccp <path>  - Optional, chaincode path. Path to the chaincode. When provided the -ccn will be used as the deployed name and not the short name of the known chaincodes.
    -cci <fcn name>  - Optional, chaincode init required function to invoke. When provided this function will be invoked after deployment of the chaincode and will define the chaincode as initialization required.
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -cai <ca_imagetag> - the image tag to be used for CA (defaults to "latest")
    -verbose - verbose mode
    -h - print this message

 Possible Mode and flag combinations
   up -ca -c -r -d -s -i -verbose
   up createChannel -ca -c -r -d -s -i -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 Taking all defaults:
   network.sh up

 Examples:
   network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccl javascript  
```

在 *test-network* 目录中，运行以下命令删除先前运行的所有容器或工程：  
***./network.sh down***  
启动网络:  
***./network.sh up***   (此命令创建一个由两个对等节点和一个排序节点组成的Fabric网络)
* 运行 *./network.sh up* 时没有创建任何channel， 而是我们将在后面的步骤实现  

执行成功后 ：  
```bash  
Creating network "net_test" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating peer0.org1.example.com ... done
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                  PORTS                              NAMES
8d0c74b9d6af        hyperledger/fabric-orderer:latest   "orderer"           4 seconds ago       Up Less than a second   0.0.0.0:7050->7050/tcp             orderer.example.com
ea1cf82b5b99        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up Less than a second   0.0.0.0:7051->7051/tcp             peer0.org1.example.com
cd8d9b23cb56        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up 1 second             7051/tcp, 0.0.0.0:9051->9051/tcp   peer0.org2.example.com
```

### 1.2 测试网络的组成部分  
列出计算机正在运行的Docker容器 ： ***docker ps -a***  
* **联盟 (Consortium)** :  
 包含两个联盟成员: Org1 和 Org2  
* **组织 (Organizations)** :  
Org1 和 Org2  
* **Peer 节点 (Peer Nodes)** :  
每个组织操作一个对等节点  
Org1 的 Peer 节点: peer0.org1.example.com  
Org2 的 Peer 节点: peer0.org2.example.com  
* **排序服务 (Orderer)** :  
单节点Raft排序服务，由排序组织运行  
运行的排序节点: orderer.example.com  
* **交易验证和账本 (Transaction Validation & Ledger)** :  
Peer 节点负责存储区块链账本并在进行交易之前对其进行验证  
Peer 节点运行智能合约，包含业务逻辑  
* **交易顺序共识 (Transaction Ordering Consensus)** ：  
排序服务负责对交易进行排序并达成共识  
对等节点提交交易到排序服务  
排序节点使用Raft算法达成交易顺序的共识
* **系统通道 (System Channel)** :  
定义网络的功能，如区块制作和Fabric版本  
定义联盟成员组织

### 1.3 创建通道  
使用 *network.sh* 脚本在Org1和Org2之间创建通道并加入他们的对等节点 ：  
***./network.sh createChannel***  
成功标志 ： ***Channel 'mychannel' joined***  

可以使用 *channel* 标志创建具有自定义名称的通道：  
***./network.sh createChannel -c channel1***  
* 通道标志还允许您创建多个不同名称的多个通道  

一步建立网络并创建频道，则可以使用 *up* 和 *createChannel* 模式一起 ：  
***./network.sh up createChannel***  
### 1.4 在通道启动一个链码  
* ***创建通道 (Channel Creation)*** :  
通道是区块链网络中的一个概念，它允许不同的组织在共享的账本上进行交易，同时保持各自的隐私  
通道一旦创建，参与的组织可以在该通道上共同执行智能合约
* ***部署智能合约 (Smart Contract Deployment)*** :  
智能合约以链码的形式部署在网络上  
链码是一个包含业务逻辑的软件包，它定义了如何与账本交互以及如何处理交易  
在将链码部署到通道之前，通道的成员必须就链码的定义达成共识  
* ***背书政策 (Endorsement Policy)*** :  
背书政策定义了交易需要获得多少个组织的背书（签名）才能被提交到账本  
这是Fabric信任模型的一部分，旨在防止任何单一组织篡改账本  
* ***执行智能合约 (Smart Contract Execution)*** :  
当交易被创建时，它需要通过智能合约执行  
每个组织在自己的对等节点上执行智能合约，并根据交易数据生成输出  
* ***交易背书 (Transaction Endorsement)*** :  
每个组织对交易输出进行签名，这个过程称为背书  
背书后的交易包含来自不同组织的签名，证明了交易的有效性  
* ***提交交易 (Transaction Submission)*** :  
如果交易获得了足够的背书，它就可以被提交到通道的账本上   
提交交易后，交易将被记录在区块链上，并且对通道中的所有组织可见  
* ***账本交互 (Ledger Interaction)*** :  
智能合约定义了如何创建、读取、写入和删除账本上的资产  
应用程序通过调用智能合约与账本交互，执行如资产转移等操作
* ***链码治理 (Chaincode Governance)*** :  
在通道中部署链码之前，需要通道成员就链码的治理达成共识  
这包括决定谁可以修改链码以及如何进行修改  

使用 *network.sh* 创建频道后，您可以使用以下命令在通道上启动链码：  
***./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go***  

*deployCC* 子命令将在 peer0.org1.example.com 和 peer0.org2.example.com 上安装   *asset-transfer (basic)* 链码  
第一次部署一套链码，脚本将安装链码的依赖项  
默认情况下，脚本安装Go版本的 *asset-transfer (basic)* 链码  
也可以使用语言便签 *-ccl* ，用于安装 Java 或 javascript 版本的链码  

* 可在 *fabric-samples* 目录的 *asset-transfer-basic* 文件夹中找到 *asset-transfer (basic)* 链码  
### 1.5 与网络交互  
*peer* CLI允许调用已部署的智能合约，更新通道，或安装和部署新的智能合约  

* *fabric-samples* 代码库的 *bin* 文件夹中找到 *peer* 二进制文件  
以下命令将这些二进制文件添加到自己的CLI路径：  

***export PATH=${PWD}/../bin:$PATH***  

还需要将 *fabric-samples* 代码库中的 *FABRIC_CFG_PATH* 设置为指向其中的 *core.yaml* 文件：  
***export FABRIC_CFG_PATH=$PWD/../config/***  

可以设置环境变量，以允许操作者作为 *Org1 *操作 *peer* CLI ：   
```bash  
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```   
运行以下命令用一些资产来初始化账本 ：  
```bash  
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'  
```  
当一个网络成员希望在账本上转一些或者改变一些资产，链码会被调用  
使用以下的指令来通过调用 *asset-transfer (basic)* 链码改变账本上的资产所有者：  
```bash
  peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```  
可以使用另一个查询来查看调用如何改变了区块链账本的资产  
可以把这个查询链码的机会通过 Org2 的 *peer* 来运行  
设置以下的环境变量来操作 Org2 ：  
```bash  
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

以下指令可获取添加到通道账本的资产列表 ：  
***peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'***  
### 1.6 使用认证机构建立网络  
默认情况下，脚本使用cryptogen工具创建证书和密钥  
该工具用于开发和测试，并且可以快速为具有有效根信任的 *Fabric* 组织创建所需的加密材料  

测试网络脚本还提供了使用证书颁发机构（CA）的网络的启动选项  
在产品网络中，每个组织操作一个CA（或多个中间CA）来创建属于他们的组织身份  
所有由该组织运行的CA创建的身份享有相同的组织信任根源  
使用CA建立测试网络，提供了在产品中部署网络的指导  
部署CA还可以让您注册Fabric SDK的客户端身份，并为您的应用程序创建证书和私钥  

先关闭现在使用的所有网络  
可以使用 *CA* 标志启动网络：  
***./network.sh up -ca***  

### 1.7 故障排除  
* 可以使用以下命令删除先前运行的工件，加密材料，容器，卷和链码镜像 ：  
***./network.sh down***  
* Docker错误，请先检查您的Docker版本(Prerequisites)， 然后尝试重新启动Docker进程  
Docker的问题是经常无法立即识别的  
如果问题仍然存在，则可以删除镜像并从头开始 ：  
***docker rm -f $(docker ps -aq)*** 

  ***docker rmi -f $(docker images -q)***  
* 如果在创建，批准，提交，调用或查询命令时发现错误，确保已正确更新通道名称和链码名称  
* 看到以下错误 ：  
*Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)*  
则可能有先前运行中链码镜像，删除它们后再次尝试  
***docker rmi -f $(docker images | grep dev-peer[0-9] | awk '{print $3}')***  
* 看到以下错误 ：  
*[configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
panic: Error reading configuration: Unsupported Config Type ""*  
即没有正确设置环境变量 *FABRIC_CFG_PATH*   
返回执行 *export FABRIC_CFG_PATH=$PWD/configtx/configtx.yaml* ，然后重新创建您的通道工件  
* 看到错误消息指出您仍然具有“active endpoints”，请清理您的 *Docker* 网络 ：  
***docker network prune***  
* 运行 *./network.sh createChannel* ，可能看到下面的错误： and it fails with the following error :  
需要卸载 Docker Dekstop 然后重新安装它，推荐的版本是 2.5.0.1，然后再克隆一份  *fabric-samples* 代码，再试图运行上面的命令  
* 类似下面的错误：  
*/bin/bash: ./scripts/createChannel.sh: /bin/bash^M: bad interpreter: No such file or directory*  
确保有问题的文件（在此示例中为createChannel.sh）为以Unix格式编码  
这很可能是由于未在 Git 配置中将 *core.autocrlf* 设置为 false  
如果有例如vim编辑器，打开文件：  
***vim ./fabric-samples/test-network/scripts/createChannel.sh***  
执行以下vim命令来更改其格式：  
***:set ff=unix***  
仍然发现错误，请在fabric-questions上共享您的日志 Hyperledger Rocket chat或StackOverflow  
---  
---  
## 2. 将智能合约部署到通道  
### 2.1 启用网络  
使用以下命令进入到您本地克隆的 *fabric-samples* 仓库中的测试网络的文件目录之下 ：  
***cd fabric-samples/test-network***  
终止所有活动的或过时的 docker 容器，并删除以前生成的构件 ：  
***./network.sh down***  
启动测试网络：  
***./network.sh up createChannel·***  

使用 *Peer* CLI 将 *asset-transfer (basic)* 链码部署到通道上，步骤如下：  

* [第一步 : 打包智能合约](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/deploy_chaincode.html#package-the-smart-contract)  
* [第二步 ：安装链码包](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/deploy_chaincode.html#install-the-chaincode-package)  
* [第三步 ：批准联码定义](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/deploy_chaincode.html#approve-a-chaincode-definition)  
* [第四步 ：向通道提供链码定义](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/deploy_chaincode.html#committing-the-chaincode-definition-to-the-channel)  

### 2.2 设置 Logspout  
安装和配置 *Logspout* 的脚本是 *monitordocker.sh* ，其已被包含在 *test-network*  文件夹的 *Fabric samples* 中  
***Logspout*** 工具将会在终端中持续输出日志，因此需要使用一个新的终端窗口  

打开一个新终端并进入到 *test-network* 文件夹下 :  
***cd fabric-samples/test-network***  

使用以下命令启动 Logspout：  
***./monitordocker.sh fabric_test***  
### 2.3 打包智能合约  
#### 2.3.1 Go  
在打包链码前，我们需要安装链码依赖  
切换到 Go 语言版本的 *asset-transfer (basic)* 目录下  
***cd fabric-samples/asset-transfer-basic/chaincode-go***   
```bash
$ cat go.mod
module github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go

go 1.14

require (
        github.com/golang/protobuf v1.3.2
        github.com/hyperledger/fabric-chaincode-go v0.0.0-20200424173110-d7076418f212
        github.com/hyperledger/fabric-contract-api-go v1.1.0
        github.com/hyperledger/fabric-protos-go v0.0.0-20200424173316-dd554ba3746e
        github.com/stretchr/testify v1.5.1
)  
```

该例子使用 Go module 安装链码依赖  
依赖将会被列举到 *go.mod* 的文件中，其在 *asset-transfer-basic/chaincode-go* 的文件夹下  

*go.mod* 文件将 **Fabric** 合约 *API* 导入到智能合约包  
可用文本编辑器打开 *asset-transfer-basic/chaincode-go/chaincode/smartcontract.go* ，来查看如何使用合约 *API* 在智能合约的开头定义 *SmartContract* 类型  
```bash
// SmartContract provides functions for managing an Asset
type SmartContract struct {
 contractapi.Contract
}  
```

*SmartContract* 类型用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  
```bash
// CreateAsset issues a new asset to the world state with given details.
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
 exists, err := s.AssetExists(ctx, id)
 if err != nil {
  return err
 }
 if exists {
  return fmt.Errorf("the asset %s already exists", id)
 }

 asset := Asset{
  ID:             id,
  Color:          color,
  Size:           size,
  Owner:          owner,
  AppraisedValue: appraisedValue,
 }
 assetJSON, err := json.Marshal(asset)
 if err != nil {
  return err
 }

 return ctx.GetStub().PutState(id, assetJSON)
}
```

为了安装智能合约依赖，应在 *asset-transfer-basic/chaincode-go* 文件夹下运行以下命令：  
***GO111MODULE=on go mod vendor***  
如果命令生效，go 的依赖包将会被安装到 *vendor* 文件夹下  

回到 *test-network* 的工作目录下，然后将链码连同网络的其他构件一同打包  
***cd ../../test-network***  

> 可以使用 *peer* CLI 在创建链码包时指定所需格式  
*peer* 的二进制文件在 *bin* 目录下的 *fabric-samples* 仓库中  
以下命令将这些二进制文件添加到 CLI 路径：  
***export PATH=${PWD}/../bin:$PATH***  
> 也需通过设置 *FABRIC_CFG_PATH* 在 *fabric-samples* 仓库中指定 *core.yaml* 文件：  
***export FABRIC_CFG_PATH=$PWD/../config/***  
>版本需为2.0.0或者更新的版本，以便能够运行本教程  
***peer version***  

现在，便可以使用 *peer lifecycle chaincode package* 的命令创建链码包：  
```bash  
 peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0  
 ```

* 这个命令将会在您当前的文件夹下创建一个名为 *basic.tar.gz* 的包  
**--lang** 标记用来指明链码语言  
**--path** 标记用来提供智能合约代码的位置  
这个路径必须是完全有效的路径，或者是当前工作目录下的相对路径  
**--label** 标志用于指定一个链码标签，该标签将在安装后识别链码  
**注意** ： 建议标签包含链码名称和版本  
#### 2.3.2 JavaScript  
切换到 *JavaScript* 语言版本的 *asset-transfer (basic)* 目录下  
***cd fabric-samples/asset-transfer-basic/chaincode-javascript***  

依赖将会被列举在 *asset-transfer-basic/chaincode-javascript* 目录下的 *package.json* 文件中  
```bash
"dependencies": {
  "fabric-contract-api": "^2.0.0",
  "fabric-shim": "^2.0.0"  
```

*package.json* 文件将 *Fabric* 合约 *class* 导入到智能合约包  

 可用文本编辑器打开 *lib/assetTransfer.js* 来查看导入到智能合约及用于创建 *asset-transfer (basic)* 的合约 *class*  

 ```bash
 const { Contract } = require('fabric-contract-api');

class AssetTransfer extends Contract {
 ...
}  
```
*AssetTransfer class* 用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  
```bash
async CreateAsset(ctx, id, color, size, owner, appraisedValue) {
        const asset = {
            ID: id,
            Color: color,
            Size: size,
            Owner: owner,
            AppraisedValue: appraisedValue,
        };

        await ctx.stub.putState(id, Buffer.from(JSON.stringify(asset)));
    }  
```

为了安装智能合约依赖，在 *asset-transfer-basic/chaincode-javascript* 文件夹下运行以下命令：  
***npm install***  
如果命令生效，*JavaScript* 的依赖包将会被安装到 *node_modules* 文件夹下  

回到 *test-network* 的工作目录下，然后将链码连同网络的其他构件一同打包  
***cd ../../test-network***  

> 可以使用 *peer* CLI 在创建链码包时指定所需格式  
*peer* 的二进制文件在 *bin* 目录下的 *fabric-samples* 仓库中  
以下命令将这些二进制文件添加到 CLI 路径：  
***export PATH=${PWD}/../bin:$PATH***  
> 也需通过设置 *FABRIC_CFG_PATH* 在 *fabric-samples* 仓库中指定 *core.yaml* 文件：  
***export FABRIC_CFG_PATH=$PWD/../config/***  
>版本需为2.0.0或者更新的版本，以便能够运行本教程  
***peer version***  

现在，便可以使用 *peer lifecycle chaincode package* 的命令创建链码包：  
```bash
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_1.0  
```

* 这个命令将会在您当前的文件夹下创建一个名为 *basic.tar.gz* 的包  
**--lang** 标记用来指明链码语言  
**--path** 标记用来提供智能合约代码的位置  
这个路径必须是完全有效的路径，或者是当前工作目录下的相对路径  
**--label** 标志用于指定一个链码标签，该标签将在安装后识别链码  
**注意** ： 建议标签包含链码名称和版本  
#### 2.3.3 Typescript  
切换到 *TypeScript* 语言版本的 *asset-transfer (basic)* 目录下  
***cd fabric-samples/asset-transfer-basic/chaincode-typescript***   

依赖将会被列举在 *asset-transfer-basic/chaincode-typescript* 目录下的 *package.json* 文件中  
```bash
"dependencies": {
  "fabric-contract-api": "^2.0.0",
  "fabric-shim": "^2.0.0"
```

*package.json* 文件将 *Fabric* 合约 *class* 导入到智能合约包  
可用文本编辑器打开 *src/assetTransfer.ts* 来查看导入到智能合约及用于创建 *asset-transfer (basic)* 的合约 *class*  

* 注意 **Asset class** 是从名为 *asset.ts* 类型的定义文件中导入  
```bash
import { Context, Contract } from 'fabric-contract-api';
import { Asset } from './asset';

export class AssetTransfer extends Contract {
 ...
}
```

*AssetTransfer class* 用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  
```bash
 // CreateAsset issues a new asset to the world state with given details.
    public async CreateAsset(ctx: Context, id: string, color: string, size: number, owner: string, appraisedValue: number) {
        const asset = {
            ID: id,
            Color: color,
            Size: size,
            Owner: owner,
            AppraisedValue: appraisedValue,
        };

        await ctx.stub.putState(id, Buffer.from(JSON.stringify(asset)));
    }
```

为了安装智能合约依赖，请在 *asset-transfer-basic/chaincode-typescript* 文件夹下运行以下命令：  
***npm install***  
如果命令生效，*JavaScript* 的依赖包将会被安装到 *node_modules* 文件夹下  

回到 *test-network* 的工作目录下，然后将链码连同网络的其他构件一同打包  
***cd ../../test-network***  

> 可以使用 *peer* CLI 在创建链码包时指定所需格式  
*peer* 的二进制文件在 *bin* 目录下的 *fabric-samples* 仓库中  
以下命令将这些二进制文件添加到 CLI 路径：  
***export PATH=${PWD}/../bin:$PATH***  
> 也需通过设置 *FABRIC_CFG_PATH* 在 *fabric-samples* 仓库中指定 *core.yaml* 文件：  
***export FABRIC_CFG_PATH=$PWD/../config/***  
>版本需为2.0.0或者更新的版本，以便能够运行本教程  
***peer version***   

可使用peer lifecycle chaincode package 的命令创建链码包：  
```bash  
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-typescript/ --lang node --label basic_1.0
```

* 这个命令将会在您当前的文件夹下创建一个名为 *basic.tar.gz* 的包  
**--lang** 标记用来指明链码语言  
**--path** 标记用来提供智能合约代码的位置  
这个路径必须是完全有效的路径，或者是当前工作目录下的相对路径  
**--label** 标志用于指定一个链码标签，该标签将在安装后识别链码  
**注意** ： 建议标签包含链码名称和版本  
### 2.4 安装联盟包  
首先在 Org1 的 *peer* 上安装链码  
设置以下环境变量以 Org1 的管理员的身份去操作 *peer* CLI  
而 *CORE_PEER_ADDRESS* 将被设置为指向 Org1 的 *peer0.org1.example.com*  
```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

执行 *peer lifecycle chaincode install* 命令安装 *peer* 上的链码  
***peer lifecycle chaincode install basic.tar.gz***  
如果命令生效，*peer* 将会生成和返回包 ID   
这个包 ID 将在下一步中用于批准链码  
```bash
2020-07-16 10:09:57.534 CDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nJbasic_1.0:e2db7f693d4aa6156e652741d5606e9c5f0de9ebb88c5721cb8248c3aead8123\022\tbasic_1.0" >
2020-07-16 10:09:57.534 CDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: basic_1.0:e2db7f693d4aa6156e652741d5606e9c5f0de9ebb88c5721cb8248c3aead8123
```

然后可以在 Org2 *peer* 上安装链码  
设置以下环境变量以 Org2 的管理员的身份去操作*peer* CLI  
*CORE_PEER_ADDRESS* 将被设置为指向 Org1 的 *peer0.org2.example.com*  
```bash
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

执行以下命令去安装链码 ：  
***peer lifecycle chaincode install basic.tar.gz***  
### 2.5 批准链码定义  
在安装了链码包后，需要为组织批准链码定义  
定义包括链码管理的**重要参数，如名称、版本和链码背书策略**   

**重要步骤** ：  

* **查询已安装的链码包** :  
使用 ***peer lifecycle chaincode queryinstalled*** 命令查询本地区域中已安装的链码包   
此命令将返回已安装链码的包ID，包括名称、版本和序列号  
* **保存为环境变量** ：  
将***peer lifecycle chaincode queryinstalled***命令返回的包 ID 粘贴到以下命令中  
**注意** ： 并非所有用户的包 ID 都相同，因此您需要使用上一步命令窗口返回的包 ID 来完成此步骤  
* **批准链码定义** ：  
环境变量已经被设置为作为 Org2 管理员来进行操作  
以 Org2 的身份批准 *asset-transfer (basic)* 的链码定义  
链码在组织级别得到批准，因此命令只需针对一个 *peer*  
批准会被在组织的内部通过 *gossip* 分发给其他 *peer*  
使用 ***peer lifecycle chaincode approveformyorg*** 命令批准链码定义  
* **以管理员角色批准** ：  
*CORE_PEER_MSPCONFIGPATH* 变量需要指向包含管理员身份的 MSP 文件  
将批准提交给排序服务，该服务将验证管理员签名，然后将批准分发给自己的 *peers*  
### 2.6 提交链码定义到通道  
可以使用 ***peer lifecycle chaincode checkcommitreadiness*** 命令来检查通道成员是否已经批准了相同的链码定义  
用于 *checkcommitreadiness* 命令的标志与用于批准组织链码的标志相同  
该命令将生成一个 JSON 映射，显示通道成员是否批准了在 *checkcommitreadiness* 命令中指定的参数  

由于作为通道成员的两个组织都批准了相同的参数，因此链码定义已准备好提交给通道  
可以使用 ***peer lifecycle chaincode commit*** 命令将链码定义提交到通道  
*commit* 命令也需要由组织管理员提交  

可以使用 ***peer lifecycle chaincode querycommitted*** 命令确认链码定义已经提交到通道  
如果 *chaincode* 被成功提交到通道，*querycommitted* 命令将返回链码定义的序列和版本  
### 2.7 调用链码  
在将链码定义提交给通道后，链码将在加入到安装了链码的通道的 *peers* 上启动  
*asset-transfer (basic)* 链码现在可以被客户端应用程序调用了  
### 2.8 升级智能合约  
在 *test-network* 目录下执行以下命令来安装链码依赖  
***cd ../asset-transfer-basic/chaincode-javascript***  
***npm install***  
***cd ../../test-network***  
可以执行以下命令从 *test-network* 目录打包 JavaScript 链码 ：

```bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer lifecycle chaincode package basic_2.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_2.0
```
运行以下命令以 Org1 管理员的身份操作peer CLI :  
```bash  
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```  
可以使用以下命令在 Org1 peer 上安装新的链码包 :  
```bash  
peer lifecycle chaincode install basic_2.tar.gz
```  
新的链码包将创建一个新的包 ID  
可以通过查询 *peer* 来查找新的包 ID :  
```TypeScript  
peer lifecycle chaincode queryinstalled
```  

* 可以使用包标签查找新链码的包 ID，并将其保存为新的环境变量  
* 每个链码包输出的包 ID 都会不同，所以不要复制和粘贴  

Org1 现在可以批准新的链码定义 ：  
```TypeScript
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```  
现在需要安装链码包，并批准链码定义为 Org2，以便升级链码  
执行以下命令以 Org2 管理员的身份操作 *peer* CLI :  
```TypeScript  
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```  
我们可以使用以下命令在 Org2 *peer* 上安装新的链码包 :  
***peer lifecycle chaincode install basic_2.tar.gz***  

使用 *peer lifecycle chaincode checkcommitreadiness* 命令检查序列 2 的链码定义是否准备好提交到通道 ：  
```TypeScript  
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json
```  
命令返回以下 *JSON* ，则可以升级链码 :  
```bash   
{
  "Approvals": {
    "Org1MSP": true,
    "Org2MSP": true
  }
}
```  
Org2 可以使用以下命令升级链码 :  
```TypeScript  
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```  
使用 *docker ps* 命令来验证新的链码是否已经在您的 *peers* 上启动:  
```TypeScript  
$ docker ps
CONTAINER ID        IMAGE                                                                                                                                                                    COMMAND                  CREATED             STATUS              PORTS                              NAMES
7bf2f1bf792b        dev-peer0.org1.example.com-basic_2.0-572cafd6a972a9b6aa3fa4f6a944efb6648d363c0ba4602f56bc8b3f9e66f46c-69c9e3e44ed18cafd1e58de37a70e2ec54cd49c7da0cd461fbd5e333de32879b   "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes                                           dev-peer0.org1.example.com-basic_2.0-572cafd6a972a9b6aa3fa4f6a944efb6648d363c0ba4602f56bc8b3f9e66f46c
985e0967c27a        dev-peer0.org2.example.com-basic_2.0-572cafd6a972a9b6aa3fa4f6a944efb6648d363c0ba4602f56bc8b3f9e66f46c-158e9c6a4cb51dea043461fc4d3580e7df4c74a52b41e69a25705ce85405d760   "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes                                           dev-peer0.org2.example.com-basic_2.0-572cafd6a972a9b6aa3fa4f6a944efb6648d363c0ba4602f56bc8b3f9e66f46c
31fdd19c3be7        hyperledger/fabric-peer:latest                                                                                                                                           "peer node start"        About an hour ago   Up About an hour    0.0.0.0:7051->7051/tcp             peer0.org1.example.com
1b17ff866fe0        hyperledger/fabric-peer:latest                                                                                                                                           "peer node start"        About an hour ago   Up About an hour    7051/tcp, 0.0.0.0:9051->9051/tcp   peer0.org2.example.com
4cf170c7ae9b        hyperledger/fabric-orderer:latest
```  

### 2.9 清理  
使用完链码后，还可以使用以下命令删除 *Logspout* 工具  
***docker stop logspout***  
***docker rm logspout***   

可以通过在test-network目录下执行以下命令来关闭测试网络 :  
***./network.sh down***  
### 2.10 常见难题解决  
#### 2.10.1 组织不同意使用链码
*  **问题** ：  
当我尝试向通道提交新的链码定义时，*peer lifecycle chaincode commit* 命令失败，出现以下错误 :  
```bash  
Error: failed to create signed transaction: proposal response was not successful, error code 500, msg failed to invoke backing implementation of 'CommitChaincodeDefinition': chaincode definition not agreed to by this org (Org1MSP)
```  
* **解决方案** ：  
您可以尝试通过使用 *peer lifecycle chaincode checkcommitreadiness* 命令来解决此错误，以检查哪些通道成员已经批准了您试图提交的链码定义  
  *peer lifecycle chaincode checkcommitreadiness* 将显示哪些组织未批准您试图提交的链码定义  

#### 2.10.2 调用失败  
* **问题** ：  
第一次调用出现问题，出现以下错误 ：  
```bash  
Error: endorsement failure during invoke. response: status:500 message:"make sure the chaincode asset-transfer (basic) has been successfully defined on channel mychannel and try again: chaincode definition for 'asset-transfer (basic)' exists, but chaincode is not installed"
```  
* **解决方案** ：  
当您批准您的链码定义时可能没有设置正确的 *--package-id*  
如果正在运行一个基于 docker 的网络，可以使用docker ps命令来检查您的链码是否正在运行:  
```TypeScript  
docker ps
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS              PORTS                              NAMES
7fe1ae0a69fa        hyperledger/fabric-orderer:latest   "orderer"           5 minutes ago       Up 4 minutes        0.0.0.0:7050->7050/tcp             orderer.example.com
2b9c684bd07e        hyperledger/fabric-peer:latest      "peer node start"   5 minutes ago       Up 4 minutes        0.0.0.0:7051->7051/tcp             peer0.org1.example.com
39a3e41b2573        hyperledger/fabric-peer:latest      "peer node start"   5 minutes ago       Up 4 minutes        7051/tcp, 0.0.0.0:9051->9051/tcp   peer0.org2.example.com
```  
如果没有看到列出任何链码容器，使用 *peer lifecycle chaincode approveformyorg* 命令批准具有正确包 ID 的链码定义  
### 2.11 背书策略失败  
**问题** ：  
出现以下错误时 ：  
```TypeScript  
2020-04-07 20:08:23.306 EDT [chaincodeCmd] ClientWait -> INFO 001 txid [5f569e50ae58efa6261c4ad93180d49ac85ec29a07b58f576405b826a8213aeb] committed with status (ENDORSEMENT_POLICY_FAILURE) at localhost:7051
Error: transaction invalidated with status (ENDORSEMENT_POLICY_FAILURE)
```  
**解决方案** ：  
这个错误是提交交易没有收集到足够的背书来符合生命周期背书策略的结果  

* 可能是交易没有足够的 *peers* 来满足策略  
* 可能是由于一些 *peer* 组织没有包含 Endorsement  

通道需要包含新的 */Channel/Application/LifecycleEndorsement* 和 */Channel/Application/Endorsement* 策略  

---  
---  
## 3. 编写一个Fabric应用  
### 总纲  
* **搭建链块网络** ：
应用程序需要与区块链网络互联，需要启动基础网络和部署一个智能合约来给应用程序使用  
![图片](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/_images/AppConceptsOverview.png)  
* **运行示例和智能合约互动** ：  
 我们的应用将使用 *assetTransfer* 智能合约在账本上创建、查询和更新资产  
 我们将逐步分析应用程序的代码以及它所调用的交易, 包括创建一些初始资产、查询一个资产、查询一系列资产、创建新资产以及将资产转让给新所有者  

### 3.1 启动区块链网络  
本地 ‘’fabric-samples’’代码库的本地拷贝里，导航到 ‘’test-network’’子目录  
***cd fabric-samples/test-network***  

若已经存在一个测试网络，需要将它关闭  
然后启动 ：
***./network.sh up createChannel -c mychannel -ca***  
#### 3.1.1 部署智能合约  
通过调用 *./network.sh* 脚本并提供链码名称和语言选项来部署包含智能合约的链码包  
```bash  
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-typescript/ -ccl typescript
```  
#### 3.1.2 准备样例应用  
打开一个新的终端，然后导航到*application-gateway-typescript*目录  
***cd asset-transfer-basic/application-gateway-typescript***  
然后安装  
***npm install***  

使用 *ls* 命令后，可见目录为 ：  
```TypeScript  
dist
node_modules
package-lock.json
package.json
src
tsconfig.json
```  
### 3.2 运行样例应用  
从*asset-transfer-basic/application-gateway-typescript*目录,运行以下命令 :  
***npm start***  

* **建立与Gateway的gRPC连接**  
为了成功建立 TLS 连接,客户端使用的终端地址必须与网关的TLS证书中的地址匹配  
由于客户端访问网关的 Docker 容器时使用的是 localhost 地址, 因此需要指定一个 gRPC 选项,强制将此终端地址解释为网关的配置主机名  
```TypeScript  
const peerEndpoint = 'localhost:7051';

async function newGrpcConnection(): Promise<grpc.Client> {
    const tlsRootCert = await fs.readFile(tlsCertPath);
    const tlsCredentials = grpc.credentials.createSsl(tlsRootCert);
    return new grpc.Client(peerEndpoint, tlsCredentials, {
        'grpc.ssl_target_name_override': 'peer0.org1.example.com',
    });
}
```  
* **创建Gateway连接**  
*Gateway* 连接具有三个要求:  
> 与 Fabric Gateway 的 gRPC 连接  
用于与网络交互的客户身份  
用于为客户身份生成数字签名的签名实现  

(示例应用程序使用 Org1 用户的 X.509 证书作为客户身份，以及基于该用户的私钥的签名实现)  
```TypeScript 
const client = await newGrpcConnection();

const gateway = connect({
    client,
    identity: await newIdentity(),
    signer: await newSigner(),
});

async function newIdentity(): Promise<Identity> {
    const credentials = await fs.readFile(certPath);
    return { mspId: 'Org1MSP', credentials };
}

async function newSigner(): Promise<Signer> {
    const privateKeyPem = await fs.readFile(keyPath);
    const privateKey = crypto.createPrivateKey(privateKeyPem);
    return signers.newPrivateKeySigner(privateKey);
}
```  
* **访问要调用的智能合约**   
示例应用程序使用 Gateway 连接获取对Network的引用,然后获取该网络上部署的默认Contract  
```TypeScript  
const network = gateway.getNetwork(channelName);
const contract = network.getContract(chaincodeName);
```  
* **使用样本资产填充账本**  
应用程序使用 submitTransaction() 来调用 InitLedger 事务函数, 该函数将账本填充了一些样本资产  

submitTransaction() 将使用 Fabric Gateway 来执行以下操作 :  
> 对事务提案进行背书  
将已背书的事务提交到订购服务  
等待事务提交，更新账本状态  

```TypeScript  
await contract.submitTransaction(‘InitLedger’);
```  
* **调用事务函数读取和写入资产**   
1. 查询所以资产 ： *evaluateTransaction()* 将使用 Fabric Gateway 来调用事务函数并返回其结果  
示例应用程序的 GetAllAssets 调用如下：  
```TypeScript  
const resultBytes = await contract.evaluateTransaction('GetAllAssets');

const resultJson = utf8Decoder.decode(resultBytes);
const result = JSON.parse(resultJson);
console.log('*** Result:', result);
```  
2. 创建新资产  
```TypeScript  
const assetId = `asset${Date.now()}`;

await contract.submitTransaction(
    'CreateAsset',
    assetId,
    'yellow',
    '5',
    'Tom',
    '1300',
);
```  
3. 更新资产  
使用 submitAsync() 调用事务， 该调用在成功提交已背书的事务给订购服务后返回，而不是等待事务提交到账本  
```TypeScript  
const commit = await contract.submitAsync('TransferAsset', {
    arguments: [assetId, 'Saptha'],
});
const oldOwner = utf8Decoder.decode(commit.getResult());

console.log(`*** Successfully submitted transaction to transfer ownership from ${oldOwner} to Saptha`);
console.log('*** Waiting for transaction commit');

const status = await commit.getStatus();
if (!status.successful) {
    throw new Error(`Transaction ${status.transactionId} failed to commit with status code ${status.code}`);
}

console.log('*** Transaction committed successfully');
```  
4. 查询更新后的资产  
```TypeScript  
const resultBytes = await contract.evaluateTransaction('ReadAsset', assetId);

const resultJson = utf8Decoder.decode(resultBytes);
const result = JSON.parse(resultJson);
console.log('*** Result:', result);
```  
5. 处理事务错误  
submitTransaction() 的失败可能会生成多种不同类型的错误，指示错误发生在提交流程的哪个点， 并包含附加信息以使应用程序能够适当地响应  
```TypeScript  
try {
    await contract.submitTransaction(
        'UpdateAsset',
        'asset70',
        'blue',
        '5',
        'Tomoko',
        '300',
    );
    console.log('******** FAILED to return an error');
} catch (error) {
    console.log('*** Successfully caught the error: \n', error);
}
```  
### 3.3 清理  
***./network.sh down***  

---  
---  
## 4. 在Fabric中使用私有数据  
### 4.1 资产转移私有数据示例用例  
资产的公开详细信息，包括所有者的身份 --> *assetCollection*  
所有者同意的初始评估值 --> *Org1MSPPrivateCollection*  
Org2 的成员创建协议使用智能合约功能 'AgreeToTransfer' 进行交易并同意评估价值 --> *Org2MSPPrivateCollection*  
### 4.2 构建集合定义 JSON 文件  
* **name** : 集合的名称  
* **policy** : 定义了哪些组织中的 Peer 节点能够存储集合数据  
* **requiredPeerCount** : 私有数据要分发到的节点数，这是链码背书成功的条件之一  
* **maxPeerCount** : 为了数据冗余，当前背书节点将尝试向其他节点分发数据的数量  
如果当前背书节点发生故障，其他的冗余节点可以承担私有数据查询的任务  
* **blockToLive** : 对于非常敏感的信息，比如价格或者个人信息，这个值代表书库可以在私有数据库中保存的时间  
数据会在私有数据库中保存 blockToLive 个区块，之后就会被清除  
如果要永久保留，将此值设置为 0 即可  
* **memberOnlyRead** : 设置为 true 时，节点会自动强制集合中定义的成员组织内的客户端对私有数据仅拥有只读权限  
* **memberOnlyWrite** : 值 true 表示节点自动强制只有属于集合成员组织之一的客户允许对私有数据进行写访问  
* **endorsementPolicy** : 定义了需要满足的背书政策命令写入私有数据集合  
收藏级背书政策覆盖链码级别策略  
有关制定政策的更多信息定义请参阅背书策略主题  

资产转移私有数据示例包含文件 collections_config.json ，文件中定义了三个私有数据集合定义： *assetCollection 、 Org1MSPPrivateCollection、和 Org2MSPPrivateCollection*  
```TypeScript  
// collections_config.json

[
   {
   "name": "assetCollection",
   "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
   "requiredPeerCount": 1,
   "maxPeerCount": 1,
   "blockToLive":1000000,
   "memberOnlyRead": true,
   "memberOnlyWrite": true
   },
   {
   "name": "Org1MSPPrivateCollection",
   "policy": "OR('Org1MSP.member')",
   "requiredPeerCount": 0,
   "maxPeerCount": 1,
   "blockToLive":3,
   "memberOnlyRead": true,
   "memberOnlyWrite": false,
   "endorsementPolicy": {
       "signaturePolicy": "OR('Org1MSP.member')"
   }
   },
   {
   "name": "Org2MSPPrivateCollection",
   "policy": "OR('Org2MSP.member')",
   "requiredPeerCount": 0,
   "maxPeerCount": 1,
   "blockToLive":3,
   "memberOnlyRead": true,
   "memberOnlyWrite": false,
   "endorsementPolicy": {
       "signaturePolicy": "OR('Org2MSP.member')"
   }
   }
]
```  
### 4.3 使用链码 API 读写私有数据  
  资产转移私有数据示例根据数据的访问方式将私有数据分为三个单独的数据定义  
```TypeScript  
// Peers in Org1 and Org2 will have this private data in a side database
type Asset struct {
       Type  string `json:"objectType"` //Type is used to distinguish the various types of objects in state database
       ID    string `json:"assetID"`
       Color string `json:"color"`
       Size  int    `json:"size"`
       Owner string `json:"owner"`
}

// AssetPrivateDetails describes details that are private to owners

// Only peers in Org1 will have this private data in a side database
type AssetPrivateDetails struct {
       ID             string `json:"assetID"`
       AppraisedValue int    `json:"appraisedValue"`
}

// Only peers in Org2 will have this private data in a side database
type AssetPrivateDetails struct {
       ID             string `json:"assetID"`
       AppraisedValue int    `json:"appraisedValue"`
}
```  
* objectType, color, size, and owner 都存储在 AssetCollection 中  
* AppraisedValue 的资产存储在集合 *Org1MSPPrivateCollection* 或 *Org2MSPPrivateCollection* 中  

资产传输私有数据样本智能创建的所有数据智能合同存储在PDC中  
智能合约使用链码API中的 *GetPrivateData()* 和 *PutPrivateData()* 函数来读写私有数据和私有数据集合  
此私有数据存储在对等方的私有状态数据库中（与公共状态数据库分开），并通过 gossip 协议在授权的对等方之间传播  

#### 4.3.1 读取集合数据  
智能合约使用链码 API *GetPrivateData()* 在数据库中访问私有数据  
GetPrivateData() 有两个参数, **集合名(collection name)** 和 **数据键（data key）**  
集合 *assetCollection* 允许 Org1 和 Org2 的成员在侧数据库中保存私有数据，集合 *Org1MSPPrivateCollection* 只允许 Org1 在侧数据库中保存私有数据, 集合 *Org2MSPPrivateCollection* 只允许 Org2 在侧数据库中保存私有数据  

* **ReadAsset** 用来查询 assetID, color, size and owner 属性的值  
* **ReadAssetPrivateDetails** 用来查询 appraisedValue 属性的值  

#### 4.3.2 写入私有数据  
智能合约使用链码API接口 PutPrivateData() 将私有数据保存到私有数据库中  
使用集合 assetCollection 写入私有数据 assetID, color, size and owner  
使用集合 Org1MSPPrivateCollection 写入私有数据 appraisedValue  

```TypeScript  
// 创建资产通过将主要资产详细信息放置在资产集合中来创建新资产
// 两个组织都可以阅读。评估值存储在所有者组织特定的集合中。
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface) error {

    // 从瞬态地图获取新资产
    transientMap, err := ctx.GetStub().GetTransient()
    if err != nil {
        return fmt.Errorf("error getting transient: %v", err)
    }

    // 资产属性是私有的，因此它们在瞬态字段中传递，而不是在函数参数中传递
    transientAssetJSON, ok := transientMap["asset_properties"]
    if !ok {
        //打印错误到 stdout
        return fmt.Errorf("asset not found in the transient map input")
    }

    type assetTransientInput struct {
        Type           string `json:"objectType"` //Type is used to distinguish the various types of objects in state database
        ID             string `json:"assetID"`
        Color          string `json:"color"`
        Size           int    `json:"size"`
        AppraisedValue int    `json:"appraisedValue"`
    }

    var assetInput assetTransientInput
    err = json.Unmarshal(transientAssetJSON, &assetInput)
    if err != nil {
        return fmt.Errorf("failed to unmarshal JSON: %v", err)
    }

    if len(assetInput.Type) == 0 {
        return fmt.Errorf("objectType field must be a non-empty string")
    }
    if len(assetInput.ID) == 0 {
        return fmt.Errorf("assetID field must be a non-empty string")
    }
    if len(assetInput.Color) == 0 {
        return fmt.Errorf("color field must be a non-empty string")
    }
    if assetInput.Size <= 0 {
        return fmt.Errorf("size field must be a positive integer")
    }
    if assetInput.AppraisedValue <= 0 {
        return fmt.Errorf("appraisedValue field must be a positive integer")
    }

    // 检查资产是否已存在
    assetAsBytes, err := ctx.GetStub().GetPrivateData(assetCollection, assetInput.ID)
    if err != nil {
        return fmt.Errorf("failed to get asset: %v", err)
    } else if assetAsBytes != nil {
        fmt.Println("Asset already exists: " + assetInput.ID)
        return fmt.Errorf("this asset already exists: " + assetInput.ID)
    }

    // 获取提交客户端标识的 ID
    clientID, err := submittingClientIdentity(ctx)
    if err != nil {
        return err
    }

    // 验证客户端是否正在向其组织中的对等方提交请求
    // 这是为了确保来自另一个组织的客户端不会尝试读取或
    // 从此对等方写入私有数据。
    err = verifyClientOrgMatchesPeerOrg(ctx)
    if err != nil {
        return fmt.Errorf("CreateAsset cannot be performed: Error %v", err)
    }

    // 使提交客户端成为所有者
    asset := Asset{
        Type:  assetInput.Type,
        ID:    assetInput.ID,
        Color: assetInput.Color,
        Size:  assetInput.Size,
        Owner: clientID,
    }
    assetJSONasBytes, err := json.Marshal(asset)
    if err != nil {
        return fmt.Errorf("failed to marshal asset into JSON: %v", err)
    }

    // 将资产保存到私有数据收集典型的记录器，记录到结构托管 docker 容器中的 stdout/file，
    // 运行此链码查找容器名称，如 dev-peer0.org1.example.com-{chaincodename_version}-xyz
    log.Printf("CreateAsset Put: collection %v, ID %v, owner %v", assetCollection, assetInput.ID, clientID)

    err = ctx.GetStub().PutPrivateData(assetCollection, assetInput.ID, assetJSONasBytes)
    if err != nil {
        return fmt.Errorf("failed to put asset into private data collection: %v", err)
    }

    // 将资产详细信息保存到拥有组织可见的集合
    assetPrivateDetails := AssetPrivateDetails{
        ID:             assetInput.ID,
        AppraisedValue: assetInput.AppraisedValue,
    }

    assetPrivateDetailsAsBytes, err := json.Marshal(assetPrivateDetails) // marshal asset details to JSON
    if err != nil {
        return fmt.Errorf("failed to marshal into JSON: %v", err)
    }

    // 获取此组织的集合名称。
    orgCollection, err := getCollectionName(ctx)
    if err != nil {
        return fmt.Errorf("failed to infer private collection name for the org: %v", err)
    }

    // 将资产评估价值放入所有者组织特定的私人数据收集中
    log.Printf("Put: collection %v, ID %v", orgCollection, assetInput.ID)
    err = ctx.GetStub().PutPrivateData(orgCollection, assetInput.ID, assetPrivateDetailsAsBytes)
    if err != nil {
        return fmt.Errorf("failed to put asset private details: %v", err)
    }
    return nil
}
```  
### 4.4 启用网络  
首先清理之前环境  
```bash  
cd fabric-samples/test-network
./network.sh down
```  
从测试网络目录中，您可以使用以下命令启动使用证书颁发机构和 CouchDB 建立结构测试网络：  
***./network.sh up createChannel -ca -s couchdb***  
### 4.5 将私有数据智能合约部署到通道  
从测试网络目录运行以下命令 ：  
```lua  
./network.sh deployCC -ccn private -ccp ../asset-transfer-private-data/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')" -cccg ../asset-transfer-private-data/chaincode-go/collections_config.json
```  
需要传递私有数据收集定义文件的路径到以上命令  
作为将链码部署到通道的一部分，两个组织在通道上必须传递相同的专用数据收集定义作为一部分的文档 : doc:chaincode_lifecycle  

当两个组织都使用 Peer Lifecycle Chaincode Approveformyorg 命令时，链码定义包括了私有数据收集的路径，该路径使用 --collections-config 标志进行定义  
```lua  
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name private --version 1.0 --collections-config ../asset-transfer-private-data/chaincode-go/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
```  
在通道成员同意将私有数据收集作为链码定义的一部分之后，使用命令 peer lifecycle chaincode commit <commands/peerlifecycle.html#peer-lifecycle-chaincode-commit>进行数据收集并提交到通道 :  
```lua  
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name private --version 1.0 --sequence 1 --collections-config ../asset-transfer-private-data/chaincode-go/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --tls --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles $ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $ORG2_CA
```  
### 4.6 注册身份标识  
首先，我们需要设置以下环境变量以使用 Fabric CA 客户端 ：  
```bash  
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```  
将结构 CA 客户端主页设置为 Org1 CA 管理员的 MSP（此标识由测试网络脚本生成）  
```bash  
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org1.example.com/
```  
使用工具 fabric-ca-client 注册新的所有者客户端身份 ： 
```bash  
fabric-ca-client register --caname ca-org1 --id.name owner --id.secret ownerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```  
可以通过向 enroll 命令提供注册名称和密钥来生成身份证书和 MSP 文件夹 ：  
```bash  
fabric-ca-client enroll -u https://owner:ownerpw@localhost:7054 --caname ca-org1 -M "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```  
将节点 OU 配置文件复制到所有者身份 MSP 文件夹中 :  
```bash  
cp "${PWD}/organizations/peerOrganizations/org1.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp/config.yaml"
```  
将结构 CA 客户端主页设置为 Org2 CA 管理员 ：  
```bash  
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org2.example.com/
```  
可以使用工具 fabric-ca-client 注册新的所有者客户端身份 ：  
```bash  
fabric-ca-client register --caname ca-org2 --id.name buyer --id.secret buyerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```  
可以注册以生成标识 MSP 文件夹 ：  
```bash  
fabric-ca-client enroll -u https://buyer:buyerpw@localhost:8054 --caname ca-org2 -M "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```  
将节点 OU 配置文件复制到买家身份 MSP 文件夹中 :  
```bash  
cp "${PWD}/organizations/peerOrganizations/org2.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp/config.yaml"
```  
### 4.7 在私有数据中创建资产  
```bash  
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```  
使用 CreateAsset 函数创建一个存储在私有数据中的资产，assetID为 asset1 ，颜色为 green，大小为 20，appraisedValue 值为 100  
私有数据 appraisedValue 将与私有数据 assetID, color, size 是分开存储的  
运行下列命令来创建资产 ：  
```bash  
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset1\",\"color\":\"green\",\"size\":20,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```  
### 4.8 已授权等对方身份查询私有数据  
集合定义允许 Org1 和 Org2 的所有对等方在其侧数据库中拥有私有数据 assetID, color, size, and owner, 但只有 Org1 中的同级才能在其侧数据库中拥有 Org1 对其私有数据 appraisedValue 的看法  

第一个 query 命令调用 ``ReadAsset``函数，该函数传递 ``assetCollection``作为参数  
```TypeScript  
// 读取资产从集合中读取信息
func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, assetID string) (*Asset, error) {

     log.Printf("ReadAsset: collection %v, ID %v", assetCollection, assetID)
     assetJSON, err := ctx.GetStub().GetPrivateData(assetCollection, assetID) //get the asset from chaincode state
     if err != nil {
         return nil, fmt.Errorf("failed to read asset: %v", err)
     }

     //未找到资产，返回空响应
     if assetJSON == nil {
         log.Printf("%v does not exist in collection %v", assetID, assetCollection)
         return nil, nil
     }

     var asset *Asset
     err = json.Unmarshal(assetJSON, &asset)
     if err != nil {
         return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
     }

     return asset, nil

 }
```  
第二个 query 命令调用 ``ReadAssetPrivateDetails`` 并传递 ``Org1MSPPrivateDetails`` 作为参数的函数  
```TypeScript  
// 读取资产私有详细信息读取组织特定集合中的资产私有详细信息
func (s *SmartContract) ReadAssetPrivateDetails(ctx contractapi.TransactionContextInterface, collection string, assetID string) (*AssetPrivateDetails, error) {
     log.Printf("ReadAssetPrivateDetails: collection %v, ID %v", collection, assetID)
     assetDetailsJSON, err := ctx.GetStub().GetPrivateData(collection, assetID) // 从链码状态获取资产
     if err != nil {
         return nil, fmt.Errorf("failed to read asset details: %v", err)
     }
     if assetDetailsJSON == nil {
         log.Printf("AssetPrivateDetails for %v does not exist in collection %v", assetID, collection)
         return nil, nil
     }

     var assetDetails *AssetPrivateDetails
     err = json.Unmarshal(assetDetailsJSON, &assetDetails)
     if err != nil {
         return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
     }

     return assetDetails, nil
 }
```  
使用 ReadAsset 函数查询 assetCollection 集合作为 Org1 来读取创建的资产的主要详细信息：  
```bash  
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```  
查询作为 Org1 成员的 asset1 的 私有数据 appraisedValue :  
```bash  
peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```  
### 4.9 将私有数据作为未经授权的对等方进行查询  
需要操作 Org2 的用户  
#### 4.9.1 切换到 Org2 中的对等方  
```bash  
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```  
#### 4.9.2 查询私有数据 Org2 授权情况  
```bash  
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```  
#### 4.9.3 查询 Org2 无权访问的私有数据  
运行以下命令来演示资产的 appraisedValue 未存储在 Org2 对等方的 Org2MSPPrivateCollection 中：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```  
空响应表明 asset1 的私有详细信息在买家 (Org2) 的私有集合中不存在  
Org2 中的用户也无法读取 Org1 私有数据集合  
通过在集合配置文件中设置 "memberOnlyRead": true ，我们指定只有 Org1 中的客户端才能从集合中读取数据  
Org2 的用户只能看到私有数据的公共哈希值