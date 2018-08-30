#Hyperledger Fabric CA示例

Hyperledger Fabric CA示例演示了以下内容：

*如何使用Hyperledger Fabric CA客户端和服务器生成所有加密
  材料，而不是使用cryptogen。cryptogen工具不适用于
  生产环境，因为它在同一个位置生成所有私钥
  然后必须将其复制到适当的主机或容器中。此示例演示
  如何为排序节点(orderers)，对等节点(peer)，管理员(administrators)
  和最终用户(end users)生成加密材料，以便私钥永远不会离开生成它们
  的主机或容器。

*如何使用基于属性的访问控制（ABAC）。请参阅fabric-samples/chaincode/abac/abac.go和
  注意使用*github.com/hyperledger/fabric/core/chaincode/lib/cid*包来提取
  来自调用者身份的属性。仅具有*abac.init*属性值为*true*的标识
  可以成功调用*Init*函数来实例化链码(chaincode)。

##运行本示例

1.运行本示例需要以下镜像：
  *hyperledger/fabric-ca-orderer*, *hyperledger/fabric-ca-peer*, 和 *hyperledger/fabric-ca-tools*

    #### 1.1.0
    运行此示例提供的*bootstrap.sh*脚本来下载fabric-ca示例所需的镜像。

    #### 1.0.X
    这些镜像是*github.com/hyperledger/fabric-ca*的v1.1.0发布版本中的新增功能。
    要在v1.1.0之前发行版运行本示例，必须如下手工构建这些镜像：
    a）pull下来*github.com/hyperledger/fabric*和*github.com/hyperledger/fabric-ca*库的主分支;
    b）确保这些库在你的GOPATH上;
    c）运行本示例提供的*build-images.sh*脚本。

2.要运行此示例，仅只需运行*start.sh*脚本即可。 您可以根据需要连续多次执行此操作
因为* start.sh *脚本在每次开始之前都会清理。

3.要停止由*start.sh*脚本启动的容器，可以运行*stop.sh*脚本。

##理解本示例

* *fabric-samples/fabric-ca/scripts/env.sh*脚本的顶部有一些变量，
定义了此示例的名称和拓扑。您可以按照脚本中注释的说明修改这些内容，
以自定义本示例。默认情况下，有三个组织。
orderer组织是*org0*，两个对等组织是*org1*和*org2*。

*start.sh*脚本首先构建*docker-compose.yml*文件（通过调用
*makeDocker.sh*脚本）然后启动docker容器。
*data*目录为所有容器的挂载卷。
在实际场景中挂载卷不是不必须的，但本示例使用挂载卷的
原因如下：
  a）以便所有容器都可以将其日志写入公共目录
     （即*data/logs*目录）使调试更容易;
  b）按照如下所述来同步容器启动的顺序
     （例如，在*ica*容器中的中级CA必须等待
      在*rca*容器中的根CA一致地将其证书写入
      *data*目录）;
  c）客户端访问引导证书需要通过TLS连接。

在*docker-compose.yml*文件中定义的容器是按照
以下顺序启动的。

1. *rca*（根CA）容器首先启动，每个组织一个。
一个*rca*容器为一个组织的根CA运行fabric-ca-server。
根CA证书被写入*data*目录
并在中间CA必须通过TLS连接到根CA的时候使用。

2.接下来启动*ica*（中级CA）容器。
一个*ica*容器为一个组织的中级CA运行fabric-ca-server。
这些每一个容器都向相应的根CA做了认证.
中级CA证书也写入*data*目录。

3. *setup*容器向中级CA注册身份，
生成创世块以及其他安装区块链网络所需的工件。
这是由*fabric-samples/fabric-ca/scripts/setup-fabric.sh*脚本执行。
请注意管理员身份已注册带有**abac.init=true:ecert**属性
（参见此脚本的*registerPeerIdentities*函数）。这导致
管理员的认证证书（ECert）具有名为 "abac.init"值为"true"
的属性。进一步注意该示例使用的链码
要求在身份证书中包含此属性，才能调用它的Init函数。
请参阅*fabric-samples/chaincode/abac/abac.go*上的链码）。
有关基于属性的访问控制（ABAC）的更多信息，请参阅
https://github.com/hyperledger/fabric/tree/release/core/chaincode/lib/cid/README.md。

4.orderer和peer容器启动。这些容器的命名
与*data/logs*目录中的日志文件一样。

5.*run*容器启动，它运行实际的测试用例。它创建
一个通道（channel），对等节点（peers）加入通道（channel），安装和实例化链码，
并查询和调用链代码。请参阅的*main*函数
*fabric-samples/fabric-ca/scripts/run-fabric.sh* 脚本了解更多详情。

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="创意共享许可" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />这项工作是根据<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">创作共享署名4.0国际许可</a>

