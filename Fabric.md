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

该例子使用 Go module 安装链码依赖  
依赖将会被列举到 *go.mod* 的文件中，其在 *asset-transfer-basic/chaincode-go* 的文件夹下  

*go.mod* 文件将 **Fabric** 合约 *API* 导入到智能合约包  
可用文本编辑器打开 *asset-transfer-basic/chaincode-go/chaincode/smartcontract.go* ，来查看如何使用合约 *API* 在智能合约的开头定义 *SmartContract* 类型  

*SmartContract* 类型用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  

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
*peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0*  

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

*package.json* 文件将 *Fabric* 合约 *class* 导入到智能合约包  

 可用文本编辑器打开 *lib/assetTransfer.js* 来查看导入到智能合约及用于创建 *asset-transfer (basic)* 的合约 *class*  
*AssetTransfer class* 用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  

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
*peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_1.0*  

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

*package.json* 文件将 *Fabric* 合约 *class* 导入到智能合约包  
可用文本编辑器打开 *src/assetTransfer.ts* 来查看导入到智能合约及用于创建 *asset-transfer (basic)* 的合约 *class*  

* 注意 **Asset class** 是从名为 *asset.ts* 类型的定义文件中导入  

*AssetTransfer class* 用于为智能合约中定义的函数创建交易上下文，这些函数将数据读写到区块链账本  

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

执行 *eer lifecycle chaincode install* 命令安装 *peer* 上的链码  
***peer lifecycle chaincode install basic.tar.gz***  
如果命令生效，*peer* 将会生成和返回包 ID   
这个包 ID 将在下一步中用于批准链码  

然后可以在 Org2 *peer* 上安装链码  
设置以下环境变量以 Org2 的管理员的身份去操作*peer* CLI  
*CORE_PEER_ADDRESS* 将被设置为指向 Org1 的 *peer0.org2.example.com*  

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