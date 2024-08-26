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
### 4.10 转移资产  
要转移资产，买方（接收方）需要通过调用链码函数 *AgreeToTransfer* 来同意与资产所有者相同的 *appraisedValue*  
约定的值将存储在 Org2 对等方上的 Org2MSPDetailsCollection 集合中  

运行以下命令以同意 Org2 的评估值为100 ：  
```bash  
export ASSET_VALUE=$(echo -n "{\"assetID\":\"asset1\",\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"AgreeToTransfer","Args":[]}' --transient "{\"asset_value\":\"$ASSET_VALUE\"}"
```  
买方现在可以查询他们在 Org2 私有数据收集中同意的值 ：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```  
资产需要由拥有资产的身份转移， 所以让我们充当 Org1 ：  
```bash  
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```  
来自 Org1 的所有者可以读取由 *AgreeToTransfer* 交易添加的数据，以查看买方身份 ：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadTransferAgreement","Args":["asset1"]}'
```  
智能合约使用 *GetPrivateDataHash（）* 函数来检查 *Org1MSPPrivateCollection* 中资产评估值的哈希值是否与 *Org2MSPPrivateCollection* 中评估值的哈希值匹配  

* 如果哈希相同，则确认所有者和感兴趣的买方已同意相同的资产价值  
* 如果满足条件，转账函数将获取买家的客户ID从转让协议中，并使买方成为资产的新所有者  
* 转移功能还将从前所有者的收藏中删除资产评估值，以及从 *assetCollection* 中删除转让协议  

运行以下命令以传输资产  
(所有者需要提供资产 ID 和买方到转移交易的组织 MSP ID)  
```bash  
export ASSET_OWNER=$(echo -n "{\"assetID\":\"asset1\",\"buyerMSP\":\"Org2MSP\"}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"TransferAsset","Args":[]}' --transient "{\"asset_owner\":\"$ASSET_OWNER\"}" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
```  
可以查询 asset1 以查看传输结果：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```  
确认转移从 Org1 集合中删除了私有详细信息：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```  
### 4.11 清除私有数据  
切换回作为 Org2 成员运行并面向 Org2 对等方 ：  
```bash  
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```  
仍然可以查询 Org2MSPPrivateCollection 中的 appraisedValue ：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```  
需要跟踪在清除私有数据之前添加的块数  
打开一个新的终端窗口并运行以下命令以查看 Org2 对等方的私有数据日志 :  
```bash  
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```  
作为 Org2 成员的终端并运行以下命令以创建三个新资产 :  
```bash  
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset2\",\"color\":\"blue\",\"size\":30,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```  
```bash  
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset3\",\"color\":\"red\",\"size\":25,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"  
```  
```bash  
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset4\",\"color\":\"orange\",\"size\":15,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```  
返回到另一个终端并运行以下命令以确认新资产导致创建了三个新块 ：  
```bash  
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```  
appraisedValue 现已从 Org2MSPDetailsCollection 私有数据收集中清除  
可以从 Org2 终端再次发出查询以查看响应是否为空 ：  
```bash  
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```  
### 4.12 使用带有私有数据的索引  
通过将索引与链代码一起打包在 *META-INF/statedb/couchdb/collections/<collection_name>/indexes* 目录中，索引也可以应用于私有数据集合  

要将链代码部署到生产环境，建议与链代码一起定义任何索引  
以便一旦链代码安装在对等节点上并在通道上实例化，链代码和支持索引就会作为一个单元自动部署  
当指定 *--collections-config* 标志指向集合 JSON 文件的位置时，关联的索引会在通道上的链代码实例化时自动部署  
### 4.13 清理  
***./network.sh down***  

---  
---  

## 5. 使用 CouchDB  
### 简介  
*Fabric* 支持两种类型的节点状态数据库  

* **LevelDB** ： 是默认嵌入在 *peer* 节点的状态数据库  
将链码数据存为简单的键值对，仅支持键、键范围和复合键查询  
* **CouchDB** ： 是一个可选的、可替换的状态数据库  
支持将账本的数据转为 JSON 格式，并支持数据内容的富查询，而不仅仅是基于 *key* 的查询  
同样支持在链码中部署索引，以实现高效查询和对大数据集的查询  

**注意** ：为了发挥 ``CouchDB`` 的优势，即基于内容的 JSON 查询，您的数据必须以 JSON 格式存储  
在设置网络之前，必须确定使用 ``LevelDB`` 还是 ``CouchDB``   
暂不支持节点从 ``LevelDB`` 切换为 ``CouchDB``  
网络中的所有节点必须使用相同的数据库类型  
想将 JSON 和二进制数据混合使用，同样可以使用 ``CouchDB`` ，但只能基于键、键范围和复合键来查询二进制数据  
### 5.1 在 Hyperledger Fabric 中启用 CouchDB  
可以用 ``CouchDB`` 的 Docker 镜像 ``CouchDB`` ，并且建议将它和节点运行在同一服务器上  
需要为每个节点设置一个 ``CouchDB`` 容器，并且更新每个节点上的配置文件 core.yaml ，将节点指向 ```CouchDB``` 容器  
core.yaml 文件的路径必须位于环境变量 *FABRIC_CFG_PATH* 指定的目录中  

* 对于 Docker 的部署，core.yaml 已经预先配置好，位于节点容器的 FABRIC_CFG_PATH 指定的文件夹中   
当使用Docker环境时，您也可以通过传递环境变量来覆盖 core.yaml 的属性，例如通过 CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS 来设置 ``CouchDB`` 地址  
* 对于原生的二进制部署， core.yaml 包含在发布的构件中  
### 5.2 创建索引  
在这个例子中， Asset 的数据结构定义如下 ：  
```TypeScript  
type Asset struct {
        DocType        string `json:"docType"` //docType is used to distinguish the various types of objects in state database
        ID             string `json:"ID"`      //the field tags are needed to keep case from bouncing around
        Color          string `json:"color"`
        Size           int    `json:"size"`
        Owner          string `json:"owner"`
        AppraisedValue int    `json:"appraisedValue"`
}
```  
在此结构中，属性（ docType, ID, color, size, owner, appraisedValue ）定义了和资产相关的账本数据  
属性 docType 可以在链码中使用，以区分链码命名空间中需要单独查询的不同数据类型  
使用 ``CouchDB`` 时，每个链码都有自己的 ``CouchDB`` 数据库，也就是说，每个链码都有自己的键的命名空间  

在 Asset 数据结构中， docType 用来标识该 JSON 文档代表资产  
在链码命名空间中可能存在其他 JSON 文档  
``CouchDB`` JSON 查询可以检索任意 JSON 字段  

在定义用于链码查询的索引时，每个索引都必须在文本文件中定义，按照 ``CouchDB`` 索引的 JSON 格式，文件扩展名为 *.json 格式  

定义一个索引 ：  

* *fields* : 查询的字段  
* *name* : 索引名  
* *type* : 格式是 json  

以下是对字段 foo 构建的名为 foo-index 索引  
```TypeScript  
{
    "index": {
        "fields": ["foo"]
    },
    "name" : "foo-index",
    "type" : "json"
}
```  
可以把设计文档（ design document ）属性 ddoc 写在索引的定义中  
为了提高效率，索引可分组写到设计文档中，但 ``CouchDB`` 建议每个设计文档只包含一个索引  
以下是以“资产转移账本查询”为例，定义的另一种索引方式，索引名称为 indexOwner，使用多个字段 docType 和 owner ，并且包括 ddoc 属性 ：  
```TypeScript  
{
  "index":{
      "fields":["docType","owner"] // Names of the fields to be queried
  },
  "ddoc":"indexOwnerDoc", // (optional) Name of the design document in which the index will be created.
  "name":"indexOwner",
  "type":"json"
}
```  
在上边的例子中，如果未指定设计文档 indexOwnerDoc ，则在部署索引时会自动创建  
可以根据字段列表中指定的一个或多个属性，或指定属性的任意组合，来构建索引  
一个属性可以存在于同一个 docType 的多个索引中  

在下边的例子中， index1 只包含 owner 属性， index2 包含 owner 和 color 属性， index3 包含 owner 、 color 和 size 属性  
```TypeScript  
{
  "index":{
      "fields":["owner"] // Names of the fields to be queried
  },
  "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index1",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color"] // Names of the fields to be queried
  },
  "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index2",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color", "size"] // Names of the fields to be queried
  },
  "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index3",
  "type":"json"
}
```  
### 5.3 将索引添加到链码文件夹  
构建索引之后，把它放到到适当的元数据文件夹下，将其与链码一起打包部署  
可以使用 peer lifecycle chaincode 命令打包并安装链码  
JSON 索引文件必须放在链码目录的 META-INF/statedb/couchdb/indexes 路径下  

![图片](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/_images/couchdb_tutorial_pkg_example.png)  

#### 5.3.1 启动网络  
启动 Fabric 测试网络，并使用它来部署资产转移账本查询的链码  
下面的命令定位到 Fabric samples 中的目录 test-network ：  
***cd fabric-samples/test-network***  
希望从一个已知的初始状态开始操作 ：  
***./network.sh down***  
如果之前从没运行过这个教程，则需要先安装链码的依赖项，才能将其部署到网络 ：  
```bash  
cd ../asset-transfer-ledger-queries/chaincode-go
GO111MODULE=on go mod vendor
cd ../../test-network
```  
在 test-network 目录中，使用以下命令部署带有 ``CouchDB`` 的测试网络 ：  
***./network.sh up createChannel -s couchdb***  
运行这个命令会创建两个 fabric peer 节点，都使用 ``CouchDB`` 作为状态数据库, 同时也会创建一个排序节点和一个名为 mychannel 的通道  
### 5.4 部署智能合约  
运行以下命令将智能合约部署到 mychannel ：  
```bash  
./network.sh deployCC -ccn ledger -ccp ../asset-transfer-ledger-queries/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
```  
#### 5.4.1 验证部署的索引  
为了查看节点上 Docker 容器的日志，请打开一个新的终端窗口，然后运行下边的命令，并过滤日志，用于确认索引已被创建 ：  
```bash  
docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"
```  
### 5.5 查询 CouchDB 状态数据库  
#### 5.5.1 在链码中查询  
* **QueryAssets –**  
这种查询方式，可以将一个选择器 JSON 查询字符串传递到函数中  
 这类查询方式，对于需要在运行时动态创建自己的查询的客户端应用程序非常有用  
* **QueryAssetsByOwner –**  
查询逻辑已在链码中定义，但允许传入查询参数  
这类查询方式，函数接受单个查询参数，即资产所有者  
然后使用 JSON 查询语法，查询状态数据库中与 asset 的 docType 和拥有者 id 相匹配的 JSON 文档  
#### 5.5.2 使用 peer 命令运行查询  
以 Org1 的身份运行下面的命令，创建一个拥有者是 “tom” 的资产 ：  
```bash  
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n ledger -c '{"Args":["CreateAsset","asset1","blue","5","tom","35"]}'
```  
之后，查询所有属于 tom 的资产  
```bash  
// Rich Query with index name explicitly specified:
peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
有3个参数值得注意：  

* **QueryAssets**  
Assets 链码中的函数名称  
QueryAssets() 调用``getQueryResultForQueryString()``，然后将 queryString 传递给 getQueryResult() shim API, 该 API 对状态数据库执行 JSON 查询  
```TypeScript  
func (t *SimpleChaincode) QueryAssets(ctx contractapi.TransactionContextInterface, queryString string) ([]*Asset, error) {
        return getQueryResultForQueryString(ctx, queryString)
}
```  
* **{"selector":{"docType":"asset","owner":"tom"}**  
这是一个 ad hoc 选择器 字符串的示例，用来查找所有 owner 属性值为 tom 的 asset 的文档  
* **"use_index":["_design/indexOwnerDoc", "indexOwner"]**  
指定设计文档名 indexOwnerDoc 和索引名 indexOwner  
在这个示例中，查询选择器通过指定 use_index 关键字显式包含了索引名  
在 ``CouchDB`` 中，如果您想在查询中显式包含索引名，则在索引定义中必须包含 ddoc 值，然后它才可以被 use_index 关键字引用  
### 5.6 使用查询和索引的最好实践  
本章节的案例有助于演示查询该如何使用索引、什么类型的查询拥有最好的性能  
**注意点** ：  

* 要查询的索引字段，必须包含在查询的选择器中或排序部分  
* 越复杂的查询性能越低，并且使用索引的几率也越低  
* 应该尽量避免会引起全表查询或全索引查询的操作符，比如： $or, $in and $regex  

在教程的前面章节，您已经对 assets 链码执行了下面的查询：  
```bash  
// Example one: query fully supported by the index
export CHANNEL_NAME=mychannel
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
已经为 asset 转移查询链码创建了 indexOwnerDoc 索引 ：  
```bash  
{"index":{"fields":["docType","owner"]},"ddoc":"indexOwnerDoc", "name":"indexOwner","type":"json"}
```  
查询中的字段 docType 和 owner 都已包含在索引中，这使得该查询成为一个完全受支持的查询  
因此这个查询能使用索引中的数据，不需要搜索整个数据库  

如果在上述查询中添加了额外的字段，它仍会使用索引  
但是，该查询必须扫描数据库以查找额外字段，从而导致响应时间更长  
```bash  
// Example two: query fully supported by the index with additional data
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\",\"color\":\"blue\"}, \"use_index\":[\"/indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
如果查询不包含索引中的所有字段，则查询会扫描整个数据库  
例如，下面的查询搜索所有者 owner，但没有指定该项拥有的类型  
```bash  
// Example three: query not supported by the index
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
由于索引 ownerIndexDoc 包含两个字段 owner 和 docType ，所以该查询不会使用索引  

$or, $in 和 $regex 等运算符通常会使得查询搜索整个索引，或者根本不使用索引  
下面的查询包含了 $or 运算符，使得查询会搜索 tom 拥有的每个资产及每个项目 :  
```bash  
// Example four: query with $or supported by the index
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"$or\":[{\"docType\":\"asset\"},{\"owner\":\"tom\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
这个查询仍然会使用索引，因为它查找的字段都包含在索引 indexOwnerDoc 中  

下面是索引不支持的复杂查询的一个例子  
```bash  
// Example five: Query with $or not supported by the index
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"$or\":[{\"docType\":\"asset\",\"owner\":\"tom\"},{\"color\":\"yellow\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```  
这个查询搜索 tom 拥有的所有资产，或颜色是黄色的其他项目  
这个查询不会使用索引，因为它需要查找整个表来匹配条件 $or  
### 5.7 在 CouchDB 状态数据库查询中使用分页  
使用 **Asset transfer ledger queries sample** 中的函数 *QueryAssetsWithPagination* 来演示在链码和客户端应用程序中如何使用分页  

本例假设已经按照上面的样例添加了 asset1
在节点的容器中，运行以下命令创建另外四个 “tom” 拥有的资产，这样 “tom” 共拥有五项资产 ：  
```bash  
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n ledger -c '{"Args":["CreateAsset","asset2","yellow","5","tom","35"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n ledger -c '{"Args":["CreateAsset","asset3","green","6","tom","20"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n ledger -c '{"Args":["CreateAsset","asset4","purple","7","tom","20"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n ledger -c '{"Args":["CreateAsset","asset5","blue","8","tom","40"]}'
```  
QueryAssetsWithPagination 增加了 pagesize 和 bookmark  
* **PageSize** 指定了每次查询返回结果的数量  
* **bookmark** 是一个 “锚（anchor）”，用来告诉 ``CouchDB`` 当前页从哪开始  

正如下面的链码函数中所示，QueryAssetsWithPagination() 调用 getQueryResultForQueryStringWithPagination() 函数，将 queryString 、bookmark 和 pagesize 传递给 GetQueryResultWithPagination()  
```TypeScript  
func (t *SimpleChaincode) QueryAssetsWithPagination(
        ctx contractapi.TransactionContextInterface,
        queryString,
        pageSize int,
        bookmark string) (*PaginatedQueryResult, error) {

        return getQueryResultForQueryStringWithPagination(ctx, queryString, int32(pageSize), bookmark)
}
```  
  
```bash  
// Rich Query with index name explicitly specified and a page size of 3:
peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3",""]}'
```  
下边是接收到的响应（为清楚起见，增加了换行），返回了5个资产中的3个，因为 pagesize 设置成了 3 :  
```bash  
{
  "records":[
    {"docType":"asset","ID":"asset1","color":"blue","size":5,"owner":"tom","appraisedValue":35},
    {"docType":"asset","ID":"asset2","color":"yellow","size":5,"owner":"tom","appraisedValue":35},
    {"docType":"asset","ID":"asset3","color":"green","size":6,"owner":"tom","appraisedValue":20}],
  "fetchedRecordsCount":3,
  "bookmark":"g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5YYg2Q5YLI5IPUgSVawJIjFXJKfm5UFANozE8s"
}
```  

```bash  
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5YYg2Q5YLI5IPUgSVawJIjFXJKfm5UFANozE8s"]}'
```  
下边是接收到的响应（为清楚起见，增加了换行），返回了5个资产中的3个，返回了剩下的2个记录：  
```bash  
{
  "records":[
    {"docType":"asset","ID":"asset4","color":"purple","size":7,"owner":"tom","appraisedValue":20},
    {"docType":"asset","ID":"asset5","color":"blue","size":8,"owner":"tom","appraisedValue":40}],
  "fetchedRecordsCount":2,
  "bookmark":"g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5aYgmQ5YLI5IPUgSVawJIjFXJKfm5UFANqBE80"
}
```  
返回的书签标记结果集的结束  
如果我们试图用这个书签进行查询，不会返回任何结果  
#### 5.7.1 范围查询分页  
**GetStateByRangeWithPagination** shim API 也会返回书签，以便应用程序在使用 LevelDB 或 CouchDB 状态数据库时可以对范围查询结果进行分页  
返回的书签代表下一个 *startKey* ，可以用于获取下一页范围查询结果  
如果所有结果都已检索完毕，返回的书签将是一个空字符串  
如果在范围查询中指定了 *endKey*，并且结果已检索完，如果使用的是 ``CouchDB`` ，返回的书签将是上次传递的 *endKey*，而如果使用的是 ``LevelDB`` ，将返回的书签则是空字符串  
### 5.8 更新索引  
为了更新索引，原来的索引定义必须包含设计文档 ddoc 属性和索引名  
要更行索引定义，请使用相同的索引名，但改变索引定义  
只需简单编辑索引 JSON 文件，并在索引中增加或者删除字段即可  
对索引名称或 ddoc 属性的更改，会导致创建新索引，但原始索引在 ``CouchDB`` 中保持不变，直到被删除  
#### 5.8.1 迭代索引定义  
在开发环境中访问 peer 节点的 ``CouchDB`` 状态数据库，则可以迭代测试各种索引以支持链码查询  
但对链码的任何改变，都需要重新部署  
使用 CouchDB Fauxton interface 或者命令行 curl 工具来创建和更新索引  

如果不想使用 Fauxton UI，下边是通过 curl 命令在 mychannel_ledger 数据库上创建索引的例子 ：   
```bash  
// Index for docType, owner.
// Example curl command line to define index in the CouchDB channel_chaincode database
 curl -i -X POST -H "Content-Type: application/json" -d
        "{\"index\":{\"fields\":[\"docType\",\"owner\"]},
          \"name\":\"indexOwner\",
          \"ddoc\":\"indexOwnerDoc\",
          \"type\":\"json\"}" http://hostname:port/mychannel_ledger/_index
```  
### 5.9 删除索引  
如果需要删除索引，就要手动使用 curl 命令或者 Fauxton 接口操作数据库  

删除索引的 curl 命令格式如下 ：   
```bash   
curl -X DELETE http://admin:adminpw@localhost:5984/{database_name}/_index/{design_doc}/json/{index_name} -H  "accept: */*" -H  "Host: localhost:5984"
```  
要删除本教程中的索引，curl 命令应该是 ：  
```bash  
curl -X DELETE http://admin:adminpw@localhost:5984/mychannel_ledger/_index/indexOwnerDoc/json/indexOwner -H  "accept: */*" -H  "Host: localhost:5984"
```  
### 5.10 清理  
***./network.sh down***  

---
---  

## 6. 创建通道  
### 6.1 创建新通道  
#### 6.1.1 配置configtxgen工具  
我们将要在 fabric-samples 目录下的 test-network 目录中进行操作  
***cd fabric-samples/test-network***  

使用以下命令将configtxgen工具添加到您的CLI路径 ：  
***export PATH=${PWD}/../bin:$PATH***  

为了使用configtxgen，您需要将FABRIC_CFG_PATH环境变量设置为本地包含configtx.yaml文件的目录的路径  
在本教程中，我们将在configtx文件夹中引用用于设置Fabric测试网络的configtx.yaml ：  
***export FABRIC_CFG_PATH=${PWD}/configtx***  

打印configtxgen帮助文本来检查是否可以使用该工具 ：  
***configtxgen --help***  
#### 6.1.2 使用configtx.yaml  
在test-network目录下的configtx文件夹中找到configtx.yaml文件，该文件用于部署测试网络  
包含以下信息，我们将使用这些信息来创建新通道 ：  

*  **Organizations** : 可以成为您的通道成员的组织  
每个组织都有对用于建立通道MSP的密钥信息的引用  
* **Ordering service** : 哪些排序节点将构成网络的排序服务，以及它们将用于同意一致交易顺序的共识方法  
该文件还包含将成为排序服务管理员的组织  
* **Channel policies** : 文件的不同部分共同定义策略，这些策略将控制组织与通道的交互方式以及哪些组织需要批准通道更新  
* **Channel profiles** : 每个通道配置文件都引用configtx.yaml文件其他部分的信息来构建通道配置  
使用预设文件来创建 Orderer 系统通道的创世块以及将被 Peer 组织使用的通道  
为了将它们与系统通道区分开来， Peer 组织使用的通道通常称为应用通道  
#### 6.1.3 启动网络  
确保您仍在本地 fabric-sample 的 est-network 目录中进行操作  
#### 6.1.4 Orderer系统通道  
执行 ./network.sh up 命令时，测试网络脚本已经创建了系统通道创世块  
创世块用于部署单个 Orderer 节点，该 Orderer 节点使用该块创建系统通道并形成网络的排序服务    
如果检查 ./ network.sh 脚本的输出，则可以在日志中找到创建创世块的命令 ：  
```bash  
configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block
```  

configtxgen 工具使用来自 configtx.yaml 的 TwoOrgsOrdererGenesis 通道配置文件来写入创世块并将其存储在 system-genesis-block 文件夹中  

下面看到TwoOrgsOrdererGenesis配置文件 ：  
```TypeScript  
TwoOrgsOrdererGenesis:
    <<: *ChannelDefaults
    Orderer:
        <<: *OrdererDefaults
        Organizations:
            - *OrdererOrg
        Capabilities:
            <<: *OrdererCapabilities
    Consortiums:
        SampleConsortium:
            Organizations:
                - *Org1
                - *Org2
```  
配置文件的 Orderer : 部分创建测试网络使用的单节点 Raft 排序服务，并以 OrdererOrg 作为排序服务管理员  
配置文件的 Consortiums 部分创建了一个名为 SampleConsortium : 的 Peer 组织的联盟  
这两个Peer组织Org1和Org2都是该联盟的成员  
因此，可以将两个组织都包含在测试网络创建的新通道中  
如果想添加另一个组织作为通道成员而又不将该组织添加到联盟中，则首先需要使用 Org1 和Org2 创建通道，然后通过更新通道配置添加该组织  
#### 6.1.5 创建应用通道  
运行以下命令为channel1创建一个创建通道的交易 ：  
```bash
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```  
*-channelID* 是将要创建的通道的名称  
通道名称必须全部为小写字母，少于250个字符，并且与正则表达式[a-z][a-z0-9.-]*匹配  
该命令使用 *-profile* 标志来引用 configtx.yaml 中的 TwoOrgsChannel : 配置文件，测试网络使用它来创建应用通道 ：  
```TypeScript
TwoOrgsChannel:
    Consortium: SampleConsortium
    <<: *ChannelDefaults
    Application:
        <<: *ApplicationDefaults
        Organizations:
            - *Org1
            - *Org2
        Capabilities:
            <<: *ApplicationCapabilities
```  
该配置文件从系统通道引用 SampleConsortium 的名称，并且包括来自该联盟的两个 Peer 组织作为通道成员  
因为系统通道用作创建应用通道的模板，所以系统通道中定义的排序节点成为新通道的默认共识者集合  
排序服务的管理员成为该通道的Orderer管理员  
可以使用通道更新在共识者者集合中添加或删除Orderer节点和Orderer组织  
我们可以使用peer CLI将通道创建交易提交给排序服务  
要使用 peer CLI，我们需要将 FABRIC_CFG_PATH 设置为 fabric-samples/config 目录中的 core.yaml 文件  
```bash  
export FABRIC_CFG_PATH=$PWD/../config/
```  
默认情况下，只有属于系统通道的联盟组织的管理员身份才能创建新通道  
发出以下命令，以Org1中的admin用户身份运行 peer CLI ：  
```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```  
以下命令创建通道 ：  
```bash  
peer channel create -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com -c channel1 -f ./channel-artifacts/channel1.tx --outputBlock ./channel-artifacts/channel1.block --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```  
上面的命令使用-f标志提供通道创建交易文件的路径，并使用-c标志指定通道名称  
-o标志用于选择将用于创建通道的排序节点  
--cafile 是 Orderer 节点的 TLS 证书的路径  
#### 6.1.6 Peer加入通道  
属于该通道成员的组织可以使用 peer channel fetch 命令从排序服务中获取通道创世块  
然后，组织可以使用创世块，通过 peer channel join 命令将 Peer 加入到该通道  
一旦 Peer 加入通道，Peer 将通过从排序服务中获取通道上的其他区块来构建区块链账本  

使用以下命令将Org1的Peer加入通道 :  
```bash  
peer channel join -b ./channel-artifacts/channel1.block
```  
使用 peer channel getinfo 命令验证 Peer 是否已加入通道 ：  
```bash  
peer channel getinfo -c channel1
```  
该命令将列出通道的区块高度和最新区块的哈希  
由于创世块是通道上的唯一区块，因此通道的高度将为 1 ：  
```bash  
2020-03-13 10:50:06.978 EDT [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
Blockchain info: {"height":1,"currentBlockHash":"kvtQYYEL2tz0kDCNttPFNC4e6HVUFOGMTIDxZ+DeNQM="}
```  
以 Org2 管理员的身份运行 peer CLI :  
```bash  
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```  
尽管我们的文件系统上仍然有通道创世块，但在更现实的场景下，Org2 将从排序服务中获取块  
```bash
peer channel fetch 0 ./channel-artifacts/channel_org2.block -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```  
该命令使用 0 来指定它需要获取加入通道所需的创世块  
该命令返回通道生成块并将其命名为 channel_org2.block  ，以将其与由 Org1 拉取的块区分开
可以使用该块将 Org2 的 Peer 加入该通道 ：  
```bash  
peer channel join -b ./channel-artifacts/channel_org2.block
```  
#### 6.1.7 配置锚节点  
组织的Peer加入通道后，应至少选择一个 Peer 成为锚定节点  
为了利用诸如私有数据和服务发现之类的功能，需要 Peer 锚节点  
每个组织都应在一个通道上设置多个锚节点以实现冗余  

第一步是使用 peer channel fetch 命令来获取最新的通道配置块  
设置以下环境变量，以 Org1 管理员身份运行 peer CLI ：  
```bash  
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051 
```  
可以使用以下命令来获取通道配置 ：  
```bash  
peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```  
由于最新的通道配置块是通道创世块，因此将看到该通道的命令返回块 0  

通道配置块存储在channel-artifacts文件夹中，以使更新过程与其他工件分开 :  
```bash  
cd channel-artifacts
```  
使用configtxlator工具开始通道配置相关工作  
第一步是将来自 protobuf 的块解码为可以读写友好的 JSON 对象  
还将去除不必要的块数据，仅保留通道配置    
```bash  
configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq .data.data[0].payload.data.config config_block.json > config.json
```  
这些命令将通道配置块转换为简化的JSON config.json，它将作为我们更新的基准  
因为我们不想直接编辑此文件，所以我们将制作一个可以编辑的副本  
```bash
cp config.json config_copy.json
```  

使用 jq 工具将 Org1 的 Peer 锚节点添加到通道配置中  
```bash  
jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json
```  
在 modified_config.json 文件中以 JSON 格式获取了通道配置的更新版本  
可以将原始和修改的通道配置都转换回 protobuf 格式，并计算它们之间的差异  
```bash  
configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb
```  
名为 channel_update.pb 的新的 protobuf 包含我们需要应用于通道配置的 Peer 锚节点更新  
可以将配置更新包装在交易 Envelope 中，以创建通道配置更新交易  
```bash
configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb
```  
然后使用最终的工件 config_update_in_envelope.pb 来更新通道  
回到test-network目录 ：  
**cd ..**  

可以通过向 peer channel update 命令提供新的通道配置来添加 Peer 锚节点  
因为正在更新仅影响 Org1 的部分通道配置，所以其他通道成员不需要批准通道更新  
```bash  
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```  
为 Org2 设置锚节点  
以Org2管理员的身份运行 peer CLI ：  
```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```  
拉取最新的通道配置块，这是该通道上的第二个块 ：  
```bash
peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

回到test-network目录  
***cd ..***  

通过执行以下命令来更新通道并设置 Org2 的 Peer 锚节点 ：  
```bash
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```  

可以通过运行 peer channel info 命令来确认通道已成功更新 ：  
```bash
peer channel getinfo -c channel1
```  

已经通过在通道创世块中添加两个通道配置块来更新通道，通道的高度将增加到 3 ：  
```bash
Blockchain info: {"height":3,"currentBlockHash":"eBpwWKTNUgnXGpaY2ojF4xeP3bWdjlPHuxiPCTIMxTk=","previousBlockHash":"DpJ8Yvkg79XHXNfdgneDb0jjQlXLb/wxuNypbfHMjas="}
```
#### 6.1.8 在新通道上部署链码  
可以使用 network.sh 脚本将 Fabcar 链码部署到任何测试网络通道  
***./network.sh deployCC -c channel1***  

运行命令后，应该在日志中看到链代码已部署到通道  
调用链码将数据添加到通道账本中，然后查询  

```bash
[{"Key":"CAR0","Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},
{"Key":"CAR1","Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},
{"Key":"CAR2","Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},
{"Key":"CAR3","Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},
{"Key":"CAR4","Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},
{"Key":"CAR5","Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},
{"Key":"CAR6","Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},
{"Key":"CAR7","Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},
{"Key":"CAR8","Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},
{"Key":"CAR9","Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
===================== Query successful on peer0.org1 on channel 'channel1' =====================    
```  
### 6.2 使用configtx.yaml创建通道配置  
使用本教程来学习如何使用 *configtx.yaml* 文件来构建存储在创世块中的初始通道配置  
测试网络使用的 *configtx.yaml* 文件位于 configtx 文件夹中  
在文本编辑器中打开该文件  
#### 6.2.1 Organizations  
通道 MSP 存储在通道配置中，并包含用于标识组织的节点，应用程序和管理员的证书  
*configtx.yaml* 文件的 *Organizations* 部分用于为通道的每个成员创建通道MSP和随附的MSP ID  
测试网络使用的 *configtx.yaml* 文件包含三个组织  
可以添加到应用程序通道的两个组织是 *Peer* 组织 Org1 和 Org2  
OrdererOrg 是一个 *Orderer* 组织，是排序服务的管理员  
在下面看到 *configtx.yaml* 的一部分，该部分定义了测试网络的Org1 ：  
```bash  
- &Org1
    # DefaultOrg defines the organization which is used in the sampleconfig
    # of the fabric.git development environment
    Name: Org1MSP

    # ID to load the MSP definition as
    ID: Org1MSP

    MSPDir: ../organizations/peerOrganizations/org1.example.com/msp

    # Policies defines the set of policies at this level of the config tree
    # For organization policies, their canonical path is usually
    #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
    Policies:
        Readers:
            Type: Signature
            Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
        Writers:
            Type: Signature
            Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
        Admins:
            Type: Signature
            Rule: "OR('Org1MSP.admin')"
        Endorsement:
            Type: Signature
            Rule: "OR('Org1MSP.peer')"

    # leave this flag set to true.
    AnchorPeers:
        # AnchorPeers defines the location of peers which can be used
        # for cross org gossip communication.  Note, this value is only
        # encoded in the genesis block in the Application section context
        - Host: peer0.org1.example.com
          Port: 7051
```  
* ``Name`` 字段是用于标识组织的非正式名称  
* `ID` 字段是组织的 MSP ID  
MSP ID 充当组织的唯一标识符，并且由通道策略引用，并包含在提交给通道的交易中  
* `MSPDir` 是组织创建的 MSP 文件夹的路径  
*configtxgen* 工具将使用此MSP文件夹来创建通道 MSP  
该 MSP 文件夹需要包含以下信息，这些信息将被传输到通道MSP并存储在通道配置中：  
1 . 一个CA根证书，为组织建立信任根 , CA根证书用于验证应用程序，节点或管理员是否属于通道成员  
2 . 来自 TLS CA 的根证书，该证书颁发了 *Peer* 节点或 *Orderer* 节点的 TLS 证书  
3 . 如果启用了 Node OU，则 MSP 文件夹需要包含一个 *config.yaml* 文件  
4 . 如果未启用 Node OU，则 MSP 需要包含一个 *admincerts* 文件夹  
* ``Policies`` 部分用于定义一组引用通道成员的签名策略  
* ``AnchorPeers`` 字段列出了组织的锚节点  
为了利用诸如私有数据和服务发现之类的功能，锚节点是必需的  
建议组织选择至少一个锚节点  
#### 6.2.2 Capabilities  
在 *configtx.yaml* 文件中，您将看到三个功能组 ：  
* ``Application`` 功能可控制 Peer 节点使用的功能，例如 Fabric 链码生命周期，并设置可以由加入通道的 Peer 运行的 Fabric 二进制文件的最低版本  
* ``Orderer`` 功能可控制 *Orderer* 节点使用的功能，例如 *Raft* 共识，并设置可通过 *Orderer*属于通道共识者集合的节点运行的 *Fabric* 二进制文件的最低版本  
* ``Channel`` 功能设置可以由 *Peer* 节点和 *Orderer* 节点运行的Fabric的最低版本  
#### 6.2.3 Application  
Application 部分定义了控制 *Peer* 组织如何与应用程序通道交互的策略  
这些策略控制需要批准链码定义或给更新通道配置的请求签名的 *Peer* 组织的数量  
这些策略还用于限制对通道资源的访问，例如写入通道账本或查询通道事件的能力  

测试网络使用 *Hyperledger Fabric* 提供的默认 *Application* 策略  
如果您使用默认策略，则所有 *Peer* 组织都将能够读取数据并将数据写入账本  
默认策略还要求大多数通道成员给通道配置更新签名，并且大多数通道成员需要批准链码定义，然后才能将链码部署到通道  
#### 6.2.4 Orderer  
每个通道配置都在通道共识者集合中包括 *Orderer* 节点  

测试网络使用 *configtx.yaml* 文件的 *Orderer* 部分来创建单节点 *Raft* 排序服务  
*OrdererType* 字段用于选择Raft作为共识类型 ：  
***OrdererType: etcdraft***  
*Raft* 排序服务由可以参与共识过程的共识者列表定义  
因为测试网络仅使用一个 *Orderer* 节点，所以共识者列表仅包含一个端点 ：  
```TypeScript
EtcdRaft :
    Consenters:
    - Host: orderer.example.com
      Port: 7050
      ClientTLSCert: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
      ServerTLSCert: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
    Addresses:
    - orderer.example.com:7050 
```  
共识者列表中的每个 Orderer 节点均由其端点地址以及其客户端和服务器 TLS 证书标识  
如果要部署多节点排序服务，则需要提供主机名，端口和每个节点使用的 TLS 证书的路径  
还需要将每个排序节点的端点地址添加到 *Addresses* 列表中  

* 使用 *BatchTimeout* 和 *BatchSize* 字段通过更改每个块的最大大小以及创建新块的频率来调整通道的延迟和吞吐量  
* *Policies* 部分创建用于管理通道共识者集合的策略  
该策略要求大多数 *Orderer* 管理员批准添加或删除 *Orderer* 节点，组织或对分块切割参数进行更新  
#### 6.2.5 Channel  
通道部分定义了用于管理最高层级通道配置的策略  
对于应用程序通道，这些策略控制哈希算法，用于创建新块的数据哈希结构以及通道功能级别  
在系统通道中，这些策略还控制Peer组织的联盟的创建或删除  
#### 6.2.6 Profiles  
*configtxgen* 工具读取 *Profiles* 部分中的通道配置文件以构建通道配置  
每个配置文件都使用 YAML 语法从文件的其他部分收集数据  
*configtxgen* 工具用此配置为应用程序通道创建通道创建交易，或为系统通道写入通道创世块  

测试网络使用的 *configtx.yaml* 包含两个通道配置文件 ``TwoOrgsOrdererGenesis`` 和``TwoOrgsChannel`` :   
``TwoOrgsOrdererGenesis`` 配置文件用于创建系统通道创世块 ：  
```TypeScript  
TwoOrgsOrdererGenesis:
    <<: *ChannelDefaults
    Orderer:
        <<: *OrdererDefaults
        Organizations:
            - *OrdererOrg
        Capabilities:
            <<: *OrdererCapabilities
    Consortiums:
        SampleConsortium:
            Organizations:
                - *Org1
                - *Org2
```  
该配置文件创建一个名为SampleConsortium的联盟，该联盟在 *configtx.yaml* 文件中包含两个 *Peer* 组织 Org1 和 Org2  
测试网络使用 ``TwoOrgsChannel`` 配置文件创建应用程序通道 ：  
```TypeScript
TwoOrgsChannel:
    Consortium: SampleConsortium
    <<: *ChannelDefaults
    Application:
        <<: *ApplicationDefaults
        Organizations:
            - *Org1
            - *Org2
        Capabilities:
            <<: *ApplicationCapabilities
```  
*TwoOrgsChannel* 提供了测试网络系统通道托管的联盟名称 *SampleConsortium*  
在 Application 部分中，来自联盟的两个组织 Org1 和 Org2 均作为通道成员包括在内  
### 6.3 通道策略  
与通道配置的其他部分不同，控制通道的策略由 *configtx.yaml* 文件的不同部分组合起来才能确定  
#### 6.3.1 签名策略  
默认情况下，每个通道成员都定义了一组引用其组织的签名策略  
当提案提交给 Peer 或交易提交给 Orderer 节点时，节点将读取附加到交易上的签名，并根据通道配置中定义的签名策略对它们进行评估  
每个签名策略都有一个规则，该规则指定了一组签名可以满足该策略的组织和身份  

可以在下面 *configtx.yaml* 中的 *Organizations* 部分中看到由 *Org1* 定义的签名策略 ：  
```TypeScript  
- &Org1

  ...

  Policies:
      Readers:
          Type: Signature
          Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
      Writers:
          Type: Signature
          Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
      Admins:
          Type: Signature
          Rule: "OR('Org1MSP.admin')"
      Endorsement:
          Type: Signature
          Rule: "OR('Org1MSP.peer')"
```  
上面的所有策略都可以通过Org1的签名来满足  
但是，每个策略列出了组织内部能够满足该策略的一组不同的角色  
**Admins** 策略只能由具有管理员角色的身份提交的交易满足，而只有具有 *peer* 的身份才能满足 Endorsement 策略  
#### 6.3.2 ImplicitMeta策略  
如果您的通道使用默认策略，则每个组织的签名策略将由通道配置中更高层级的 ImplicitMeta策略评估  
ImplicitMeta 策略不是直接评估提交给通道的签名，而是使用规则在通道配置中指定可以满足该策略的一组其他策略  

可以在下面的 *configtx.yaml* 文件的 *Application* 部分中看到定义的 ImplicitMeta 策略 ：  
```TypeScript  
Policies:
    Readers:
        Type: ImplicitMeta
        Rule: "ANY Readers"
    Writers:
        Type: ImplicitMeta
        Rule: "ANY Writers"
    Admins:
        Type: ImplicitMeta
        Rule: "MAJORITY Admins"
    LifecycleEndorsement:
        Type: ImplicitMeta
        Rule: "MAJORITY Endorsement"
    Endorsement:
        Type: ImplicitMeta
        Rule: "MAJORITY Endorsement"
```  
#### 6.3.3 通道修改策略  
通道结构由通道配置内的修改策略控制  
通道配置的每个组件都有一个修改策略，需要满足修改策略才能被通道成员更新  
每个修改策略都可以引用 ImplicitMeta 策略或签名策略  
则定义每个组织的值将引用与该组织关联的 Admins 签名策略  
定义通道成员集合的应用程序组的修改策略是  Channel/Application/Admins ImplicitMeta策略  
#### 6.3.4 通道策略和访问控制列表  
通道配置中的策略也由访问控制列表（ACLs）引用，该访问控制列表用于限制对通道使用的*Fabric* 资源的访问  
ACL 扩展了通道配置内的策略，以管理通道的进程  
例如，以下 ACL 限制了谁可以基于 /Channel/Application/Writers 策略调用链码 ：  
```bash  
# ACL policy for invoking chaincodes on peer
peer/Propose: /Channel/Application/Writers
```  
大多数默认 ACL 指向通道配置的 Application 部分中的 ImplicitMeta 策略  
为了扩展上面的示例，如果组织可以满足/Channel/Application/Writers策略，则可以调用链码  
![图片](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/_images/application-writers.png)  
#### 6.3.5 Orderer策略  
*configtx.yaml* 的 *Orderer* 部分中的 *ImplicitMeta* 策略以与 *Application* 部分管理 *Peer* 组织类似的方式来管理通道的 *Orderer* 节点   
*ImplicitMeta* 策略指向与排序服务管理员的组织相关联的签名策略  
![图片](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/_images/orderer-policies.png)  
如果使用默认策略，则需要大多数 *Orderer* 组织批准添加或删除 *Orderer* 节点  
![图片](https://hyperledger-fabric.readthedocs.io/zh-cn/latest/_images/orderer-admins.png)  
*Peer* 使用 Channel/Orderer/BlockValidation 策略来确认添加到通道的新块是由作为通道共识者集合一部分的 *Orderer* 节点生成的，并且该块未被篡改或被另一个 *Peer* 组织创建  
