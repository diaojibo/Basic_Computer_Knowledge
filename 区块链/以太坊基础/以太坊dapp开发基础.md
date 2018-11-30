## 以太坊dapp开发基础

部署合约最重要的工具有如下几种：

 - Truffle：是以太坊的开发环境、测试框架和资产通道。换句话说，它可以帮助你开发、发布和测试智能合约等等。你可以阅读有关 Truffle 套件的文档，了解更多信息。

 - Ganache：以前叫作 TestRPC，它在 TestRPC 和 Truffle 的集成后被重新命名为 Ganache。Ganache 的工作很简单：**创建一个虚拟的以太坊区块链**，并生成一些我们将在开发过程中用到的虚拟账号。

 - Mist：Mist 是一个分布式网络 apps 的浏览器，相当于是只针对 Dapps 的 Chrome 或 Firefox。目前来说，它仍然是不安全的，所以你还不能在不受信任的 dapp 中使用它。

 - 以太坊钱包：它是 Mist 的一个版本，但只启动一个 dapp ——以太坊钱包。Mist 和以太坊钱包只是 UI（用户界面）前端，我们还需要一个将我们连接到以太坊区块链的核心程序（它可以是一个真正的以太坊区块链，也可以是一个测试版的）。

- Geth：Geth 是把你连接到区块链的核心应用程序，它也可以启动一个新的区块链（在我们这个示例中，我们将创建一个本地测试网区块链），创建合约，挖掘以太币等。

#### Truffle
Truffle是针对基于以太坊的Solidity语言的一套开发框架。本身基于Javascript。

Truffle对客户端做了深度集成。开发，测试，部署一行命令都可以搞定。不用再记那么多环境地址，繁重的配置更改，及记住诸多的命令。

zwlj：也就是基于solidity和以太坊多了一层封装，让你更方便的进行开发了。

### 开始
话不多说，我们从Truffle开始，试着上手dapp的开发。

我们现用npm全局安装truffle和ganache

```
npm install -g truffle
npm install -g ganache-cli
```

这两步比较快就能完成，接下来我们创建一个truffle项目：

```
truffle init
```

![](image/truffle0.png)

可以看到会产生如上目录

我们在contracts文件夹中添加solidity代码文件，我们添加一个Wrestling.sol代码。

约100行的代码如下：

```
pragma solidity ^0.4.18;

    /**
    * Example script for the Ethereum development walkthrough
    */

contract Wrestling {
    /**
    * Our wrestlers
    */
	address public wrestler1;
	address public wrestler2;

	bool public wrestler1Played;
	bool public wrestler2Played;

	uint private wrestler1Deposit;
	uint private wrestler2Deposit;

	bool public gameFinished;
    address public theWinner;
    uint gains;

    /**
    * The logs that will be emitted in every step of the contract's life cycle
    */
	event WrestlingStartsEvent(address wrestler1, address wrestler2);
	event EndOfRoundEvent(uint wrestler1Deposit, uint wrestler2Deposit);
	event EndOfWrestlingEvent(address winner, uint gains);

    /**
    * The contract constructor
    */
	constructor() public {
		wrestler1 = msg.sender;
	}

    /**
    * A second wrestler can register as an opponent
    */
	function registerAsAnOpponent() public {
        require(wrestler2 == address(0));

        wrestler2 = msg.sender;

        emit WrestlingStartsEvent(wrestler1, wrestler2);
    }

    /**
    * Every round a player can put a sum of ether, if one of the player put in twice or
    * more the money (in total) than the other did, the first wins
    */
    function wrestle() public payable {
    	require(!gameFinished && (msg.sender == wrestler1 || msg.sender == wrestler2));

    	if(msg.sender == wrestler1) {
    		require(wrestler1Played == false);
    		wrestler1Played = true;
    		wrestler1Deposit = wrestler1Deposit + msg.value;
    	} else {
    		require(wrestler2Played == false);
    		wrestler2Played = true;
    		wrestler2Deposit = wrestler2Deposit + msg.value;
    	}
    	if(wrestler1Played && wrestler2Played) {
    		if(wrestler1Deposit >= wrestler2Deposit * 2) {
    			endOfGame(wrestler1);
    		} else if (wrestler2Deposit >= wrestler1Deposit * 2) {
    			endOfGame(wrestler2);
    		} else {
                endOfRound();
    		}
    	}
    }

    function endOfRound() internal {
    	wrestler1Played = false;
    	wrestler2Played = false;

    	emit EndOfRoundEvent(wrestler1Deposit, wrestler2Deposit);
    }

    function endOfGame(address winner) internal {
        gameFinished = true;
        theWinner = winner;

        gains = wrestler1Deposit + wrestler2Deposit;
        emit EndOfWrestlingEvent(winner, gains);
    }

    /**
    * The withdraw function, following the withdraw pattern shown and explained here:
    * http://solidity.readthedocs.io/en/develop/common-patterns.html#withdrawal-from-contracts
    */
    function withdraw() public {
        require(gameFinished && theWinner == msg.sender);

        uint amount = gains;

        gains = 0;
        msg.sender.transfer(amount);
    }
}
```

耐心理解一下这段代码：这个代码描写了一个wrestling游戏。大致就是两个斗钱多，双方各拿出一笔钱，如果一方是另一方的两倍以上，就是那个人赢了。

根据solidity的语法，合约对象里先是定义了一堆公有的变量(可以公共读取)。wresteler1(这个变量代表合约创建者其实)，wresteler2是挑战者。在构造函数里，合约首先把wrestler1这个变量设置为合约创建者的address，从msg这个全局变量里获得。构造函数只会在创建这个合约的时候才会被调用。

wrestler2通过register函数注册，注意register函数里的require函数。一旦有人已经注册成为了wrestler2，则回滚操作。address初始化的时候是0，如果address不为0了，说明已经被人注册了。register最后还触发了strart event事件，**这里的事件只是方便我们最后追踪用的，我们可以在客户端监听这些事件**。

最后就不难理解核心的wrestle函数了，这是游戏的核心。函数后加了payable，说明这个函数调用时允许发送一定数量的eth金额。调用这个函数以后会先判断游戏是否结束，以及是否游戏参与人进行的调用。最后判断是否两个人都注入了金额，最后判断大小，进入endofgame判定。

withdraw则是取钱函数，只有胜利者才能成功触发。

### 参考

[教程 | 以太坊开发演练，Part-2：Truffle，Ganache，Geth 和 Mist](https://ethfans.org/posts/ethereum-development-walkthrough-part-2)
