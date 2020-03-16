## chaincode教程
fabric里的智能合约，也就是chaincode主要是用go或者node编写的，这里就暂且选用go语言来写chaincode。

每个chaincode程序必须实现Chaincode接口，对于go语言，那就是继承chaincode的type struct。

开始写智能合约之前，我们先import下面2个包：

``` go
import (
	"fmt"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	"github.com/hyperledger/fabric/protos/peer"
)
```

fmt就不说了，另外两个是写智能合约必须要用到的包。然后我们定义我们的智能合约对象,学习go语言的时候，我们已经知道go里没有标准的class对象，有interface的存在，但是只要是类实现了这个interface的方法，就可以用这个interface来定义(见go语言笔记)

所以没有什么顾虑，我们直接开一个interface实现对应方法就好，注意实现智能合约接口要实现两个方法：

 - Init方法
 - Invoke方法

Init方法会在智能合约部署初始化，或者升级部署的时候调用。Invoke函数当然是响应交易时被调用，使得交易可以正确执行。


### 写一个资产转移的智能合约

接下来我们写一个智能合约对象，并且实现它的Init和Invoke方法

Init方法会需要一个参数,类型是stub shim.ChaincodeStubInterface。

``` go
type SimpleAsset struct {
}

func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
	// 获取参数
	args := stub.GetStringArgs()
	if len(args) != 2 {
		return shim.Error("Incorrect arguments. Expecting a key and a value")
	}

	// 调用stub.PutState()来创建任何的变量或资产
	// 把键值保存到账本中
	err := stub.PutState(args[0], []byte(args[1]))
	if err != nil {
		return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
	}
	return shim.Success(nil)
}
```

如上代码所示，我们先定义了一个SimpleAsset结构体，然后在结构体外定义它的成员函数(语法详细可见go语言笔记)。

Init接受一个参数，然后我们调用这个stub的GetStringArgs方法，接收参数。可知我们在初始化一份智能合约的时候可以人为传递一些参数进去帮助初始化

针对我们这个资产转移的智能合约，我们希望传递进去的是一个 **键值对**。所以我们判断接收到的参数个数是否为2，是2就返回错误。

接下来我们就调用putState方法，把键值对里的值(参数2)赋给key为arg0的位置。成功就Success，失败就Error。


Init方法写完以后我们就写 **Invoke函数**


``` go
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
  fn, args := stub.GetFunctionAndParameters()
  var result string
  var err error
  if fn == "set" {
    result, err = set(stub, args)
  } else {
    result, err = get(stub, args)
  }

  if err != nil {
    return shim.Error(err.Error())
  }
  // Return the result as success payload
  return shim.Success([]byte(result))
}
```

显然在我们client调用智能合约的时候，会给出调用智能合约的函数名和参数。这些都会被智能合约获取到，并给到Invoke函数里，Invoke函数相当于也做一层转发。

如上代码会判断函数名是不是set，不是set都当做get用。然后就会去调用set方法和get方法。有错误就返回错误，没有错误就返回成功。并且把结果当做payload返回。


#### 完成get和set函数
那么接下来就该完成具体的get和set函数了。我们要通过ChaincodeStubInterface.PutState 和ChaincodeStubInterface.GetState这两个函数来访问账本的状态

``` go
// set函数会保存资产到账本中，如果这个key以及存在了，那么会用新的value覆盖老的
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
  if len(args) != 2 {
    return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
  }
  err := stub.PutState(args[0], []byte(args[1]))
  if err != nil {
    return "", fmt.Errorf("Failed to set asset: %s", args[0])
  }
  return args[1], nil
}

// 实现get函数
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
  if len(args) != 1 {
    return "", fmt.Errorf("Incorrect arguments. Expecting a key")
  }
  value, err := stub.GetState(args[0])
  if err != nil {
    return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
  }

  if value == nil {
    return "", fmt.Errorf("Asset not found: %s", args[0])
  }
  return string(value), nil
}
```

如上就是我们自定义的get和set函数，不难看出无非就是简单的调用getState和putState，并且把结果

#### 构建main函数
我们甚至能直接创建一个main函数来作为客户端调用这个智能合约

```
func main() {
  if err := shim.Start(new(SimpleAsset)); err != nil {
    fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
  }
}
```

#### dev模式进行测试
通常来说，chaincode都由一个peer节点来启动和维护，但是再dev模式下，我们用户可以自行编译启动来进行测试，这非常有用。


在docker 官方实例的案例里，我们下载了fabric-samples实例。我们现在就复用那个项目。

那个项目里有一个chaincode-docker-devmode项目文件夹，我们cd进去。

```
docker-compose -f docker-compose-simple.yaml up
```

启动这个项目网络，当然在启动之前，我们最好还是清理一下docker容器，以防冲突。

启动以后，就进入一个容器：

```

// 终端1执行，然后会看到一大堆log
docker exec -it chaincode bash
```

上面的那些代码其实都属于sacc项目，我们ls一下就能看到这个sacc项目：

```
cd sacc
go build


// 终端2执行 看到chaincode starting up然后停住
CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
```

调用install命令可以安装chaincode

```
// 以下都在终端3完成
docker exec -it cli bash

peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
```


```
peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc

peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

### 一些总结
总而言之，在cli上处理chaincode，下面一些命令是比较重要的：

```
// install
peer chaincode install -n asset_mgmt -v 1.0 -p sacc
```

-p: chaincode的路径, 必须在 GOPATH 目录下, 如 $GOPATH/src/sacc.其余的，-n是name，-v是version

install 交易的过程会将chaincode的源码以一种被称为 ChaincodeDeploymentSpec 的规定格式打包, 并把它安装在一个将要运行该chaincode的peer节点上. Channel上每个要运行chaincode的背书节点都要安装chaincode.

没有chaincode的节点, 不能影响交易的背书阶段, 因为它们不能执行chaincode, 但是他们可以在共识完成后的验证交易阶段进行验证, 并提交交易到账本上.


#### 实例化chaincode
chaincode可能会与任意数量的Channel绑定, 并在每个Channel上独立运行. 即chaincode在多少个Channel上安装并实例化并没有什么影响, 对于每个提交交易的Channel, 其状态都是独立而互不影响的.

```
peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```

该指令初始化john的状态为0. 背书策略向Org1或Org2的成员询问所有sacc处理的交易. 即, 为确保交易有效, Org1或Org2必须为调用sacc的结果签名.
