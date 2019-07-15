### <center>以太坊智能合约安全入门(二)</center>  
------------------------

在上一篇以太坊智能合约安全入门(一)中，介绍了关于智能合约的基本内容，以及在智能合约中出现次数比较的重入漏洞，这篇文章将继续为大家讲解智能合约中的整数溢出漏洞。  

## 整数溢出原理
通常来说，在编程语言里由算数问题导致的整数溢出漏洞屡见不鲜，在区块链的世界里，智能合约的Solidity语言中也存在整数溢出问题，整数溢出一般分为又分为上溢和下溢，在智能合约中出现整数溢出的类型包括三种：乘法溢出、加法溢出、减法溢出
在Solidity语言中，变量支持的整数类型步长以8递增，支持从uint8到uint256，以及int8到int256。例如，一个 uint8类型 ，只能存储在范围 0到2^8-1，也就是[0,255] 的数字，一个 uint256类型 ，只能存储在范围 0到2^256-1的数字。

在以太坊虚拟机（EVM）中为整数指定固定大小的数据类型，而且是无符号的，这意味着在以太坊虚拟机中一个整型变量只能有一定范围的数字表示，不能超过这个制定的范围。

如果试图存储 256这个数字 到一个 uint8类型中，这个256数字最终将变成 0，所以整数溢出的原理其实很简单，为了说明整数溢出原理，这里以 8 (uint8)位无符整型为例，8 位整型可表示的范围为 [0, 255]，255 在内存中存储按位存储的形式为11111111。
8 位无符整数 255 在内存中占据了 8bit 位置，若再加上 1 整体会因为进位而导致整体翻转为 0，最后导致原有的 8bit 表示的整数变为 0。
溢出简单实例演示，这里以uint256类型演示：
```solidity
pragma solidity ^0.4.25;

contract POC{
    //加法溢出
    //如果uint256 类型的变量达到了它的最大值(2**256 - 1)，如果在加上一个大于0的值便会变成0
    function add_overflow() returns (uint256 _overflow) {
        uint256 max = 2**256 - 1;
        return max + 1;
    }


	//减法溢出
	//如果uint256 类型的变量达到了它的最小值(0)，如果在减去一个小于0的值便会变成2**256-1(uin256类型的最大值)
	function sub_underflow() returns (uint256 _underflow) {
    	uint256 min = 0;
    	return min - 1;
	}
    
    //乘法溢出
	//如果uint256 类型的变量超过了它的最大值(2**256 - 1)，最后它的值就会回绕变成0
	function mul_overflow() returns (uint256 _underflow) {
    	uint256 mul = 2**255;
    	return mul * 2;
	}
}
```

例如4月25日早间，火币Pro公告，SMT项目方反馈今日凌晨发现其交易存在异常问题，经初步排查，SMT的以太坊智能合约存在漏洞。火币Pro也同期检测到TXID https://etherscan.io/tx/0x0775e55c402281e8ff24cf37d6f2079bf2a768cf7254593287b5f8a0f621fb83 的异常。受此影响，火币Pro现决定暂停所有币种的充提币业务，随后，火币Pro又发布公告称暂停SMT/USDT、SMT/BTC和SMT/ETH的交易。此外，OKEx，gate.io等交易平台也已经暂停了SMT的充提和交易。截止暂停交易，SMT在火币Pro的价格下跌近20%。
正是由于没有做整数上溢出检查，才导致了漏洞的出现，该漏洞代理的直接经济损失高达上亿元人民币，间接产生的负面影响目前无法估量。



保护措施：为了防止整数溢出的发生，一方面可以在算术逻辑前后进行验证，另一方面可以直接使用 OpenZeppelin 维护的一套智能合约函数库中的 SafeMath 来处理算术逻辑。
```solidity
pragma solidity ^0.4.25;

library SafeMath {
  function mul(uint256 a, uint256 b) internal constant returns (uint256) {
    uint256 c = a * b;
    assert(a == 0 || c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal constant returns (uint256) {
    uint256 c = a / b;
    return c;
  }

  function sub(uint256 a, uint256 b) internal constant returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal constant returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}

```
通过使用Safemath封装的加法，减法，乘法接口，使用assert方法进行判断，可以避免产生溢出漏洞。
