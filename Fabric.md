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

在 *test-network* 目录中，运行以下命令删除先前运行的所有容器或工程：  
***./network.sh down***  
启动网络:  
***./network.sh up***   (此命令创建一个由两个对等节点和一个排序节点组成的Fabric网络)
* 运行 *./network.sh up* 时没有创建任何channel， 而是我们将在后面的步骤实现  

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
以下命令将这些二进制文件添加到您的CLI路径：  

***export PATH=${PWD}/../bin:$PATH***  

还需要将 *fabric-samples* 代码库中的 *FABRIC_CFG_PATH* 设置为指向其中的 *core.yaml* 文件：  
***export FABRIC_CFG_PATH=$PWD/../config/***  

可以设置环境变量，以允许操作者作为 *Org1 *操作 *peer* CLI ：   
***#Environment variables for Org1***  

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
***./network.sh up createChannel***
