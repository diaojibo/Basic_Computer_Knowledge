## solidity笔记
笔记总结solidity官方文档中给出的一些知识点。


### 子货币
用智能合约就能简单的实现代币了：

```
pragma solidity ^0.4.21;

contract Coin {
    // 关键字“public”让这些变量可以从外部读取
    address public minter;
    mapping (address => uint) public balances;

    // 轻客户端可以通过事件针对变化作出高效的反应
    event Sent(address from, address to, uint amount);

    // 这是构造函数，只有当合约创建时运行
    function Coin() public {
        minter = msg.sender;
    }

    function mint(address receiver, uint amount) public {
        if (msg.sender != minter) return;
        balances[receiver] += amount;
    }

    function send(address receiver, uint amount) public {
        if (balances[msg.sender] < amount) return;
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
```

对于变量，关键字 public 自动生成一个函数，允许你在这个合约之外访问这个状态变量的当前值。如果没有这个关键字，其他的合约没有办法访问这个变量。

`mapping (address => uint) public balances;`这一行定义了一个变量balances，这个变量类型是一个哈希表。key是一个address，value是一个整形。

event Sent(address from, address to, uint amount); 这行声明了一个所谓的“事件（event）”，它会在 send 函数的最后一行被发出。用户界面（当然也包括服务器应用程序）可以监听区块链上正在发送的事件，而不会花费太多成本。一旦它被发出，监听该事件的listener都将收到通知。而所有的事件都包含了 from ， to 和 amount 三个参数。

特殊函数 Coin 是在创建合约期间运行的构造函数，不能在事后调用。 **它永久存储创建合约的人的地址**

msg (以及 tx 和 block ) 是一个神奇的全局变量，其中包含一些允许访问区块链的属性。 **msg.sender 始终是当前（外部）函数调用的来源地址**。

最后，真正被用户或其他合约所调用的，以完成本合约功能的方法是 mint 和 send。 如果 mint 被合约创建者外的其他人调用则什么也不会发生。 另一方面， send 函数可被任何人用于向他人发送币 (当然，前提是发送者拥有这些币)。

### 参考
[solidity官方文档](https://solidity-cn.readthedocs.io/zh/develop/introduction-to-smart-contracts.html)
