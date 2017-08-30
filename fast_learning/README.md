# 智能合约需要注意的要点

## 无法指定的区块
交易不能保证在下一个区块或任何未来的指定区块中发生，因为矿工不能识别交易的提交者就需要打包交易。包括函数调用、交易以及创建合约等操作都无法指定区块。

## “payload”
payload就是与交易一起发送的 `.abi` 文件字节码“数据”。

## 是否有可用的反编译器？
Solidity没有反编译器。这在某种程度上原则上是可能的，但是例如变量名称将会丢失，并且需要付出巨大的努力才能使其看起来类似于原始源代码。
字节码可以反编译为操作码，这是由几个区块链探查器提供的服务。
所以如果要由第三方使用，则块上的合同应该发布其原始源代码。

## 创建可以杀死和归还资金的合同
注意杀死合同听起来好像是一个好主意，因为“清理”总是很好，但如上所述，它并没有真正的清理。此外，如果以ether被发送到删除的合同，ether就相当于永远丢失。
所以如果要停用合同，最好通过 `disable` 使所有函数抛出内部状态来禁用它们。这将使得无法使用合约，并且发送到合约的以太币将自动退回。

## solidity不能表示的数据类型
`double` 和 `float` 暂时不被支持。

## 是否可能像这样在线初始化数组： `string[] myarray = ["a", "b"];`
可以这样做，但是应该注意的是，目前这种方法只适用于静态大小的存储器阵列。你甚至可以在return语句中创建一个内联的内存数组。例如：
```
pragma solidity ^0.4.0;

contract C {
    function f() returns (uint8[5]) {
        string[4] memory adaArr = ["This", "is", "an", "array"];
        return ([1, 2, 3, 4, 5]);
    }
}
```
## 时间戳（`now`, `block.timestamp`）
由于这些时间戳函数的值来源于矿工，所以它们不一定会很可靠。假设每人破坏计算机上的时间，也就是计算机时间完全正确，可以得出以下结论：发布交易的时间 `X` 、交易包含代码里的 `now()` 函数的值 `Y` 、交易被确定在区块中的时间 `Z` ，它们三者之间的关系是 `X <= Y <= Z` 。    

永远不要使用 `now` 或 `block.hash` 作为随机来源（随机种子），除非你知道你在做什么（需求和解决方案明确）！

## 返回 `struct`
只有 `internal` 内部函数调用才可以返回 `struct` 。

## 如果我返回 `enum` ，我只在web3.js.中获得整数值。如何获取命名值？
枚举暂不支持ABI，它们只是由Solidity支持。目前你必须自己做映射，官方以后可能会提供一些帮助。

## 回退函数（未命名函数） `function () { ... }`
这个函数被称为“回退函数”，当有人恰好将Ether发送到合同而不提供任何数据（也就是给合约地址发送单纯的交易）或者有人弄乱了类型，以致于他们尝试调用不存在的函数时，它就会被调用。
在这些情况下，默认行为（如果没有明确给出回退函数）是抛出异常。
如果合同旨在通过简单的转移接收Ether，那么您应该执行回退函数
`function() payable { }`
因此，回退功能的另一个用途是，例如通过使用事件注册使合约可以接收Ether。
回退函数不能接收参数。
在特殊情况下也可以发送数据，如果没有其他函数被调用，回退函数可以通过 `msg.data` 访问交易包含的数据。

## 函数和合约如何接受Ether
如上一条，合约可以通过 `function() payable { }` 专门设置接收以太币，其中 `payable` 是solidity提供的一种装饰器，为函数提供可以接受以太币的功能。因此，调用函数时发送的交易value不为0的话，这个函数必须由 `payable` 装饰，否则，合约不会收到这些value的以太币，创建合约时（也就是区块链自动调用构造函数）除外。

## 状态变量可以在线（在创建合约时）初始化吗？
是的，这对于所有类型（甚至对于结构体）都是可能的。但是，对于数组，应该注意，你必须将其声明为静态内存数组。
```
pragma solidity ^0.4.0;

contract C {
    struct S {
        uint a;
        uint b;
    }

    S public x = S(1, 2);
    string name = "Ada";
    string[4] adaArr = ["This", "is", "an", "array"];
}

contract D {
    C c = new C();
}
```

## 如何使用循环
solidity循环与JavaScript非常相似，但是要注意：如果你使用 `for (var i = 0; i < a.length; i ++) { a[i] = i; }` 语句，其中 `i` 将从 `0` 开始计数并且类型是 `unit8` ，这意味着 `a` 如果有超过255个元素就会进入死循环，因为 `i` 最大值是 `255` 。最好还是用 `for (uint i = 0; i < a.length...` 。

## solidity用哪种字符集
尽管 `UTF-8` 是被推荐的，但是solidity源码中字符串与字符集没有关系。不过（变量、函数等的）标识符只能使用 `ASCII` 字符集。

## 字符串处理（待续）
### What are some examples of basic string manipulation (substring, indexOf, charAt, etc)?
There are some string utility functions at stringUtils.sol which will be extended in the future. In addition, Arachnid has written solidity-stringutils.

For now, if you want to modify a string (even when you only want to know its length), you should always convert it to a bytes first:
```
pragma solidity ^0.4.0;

contract C {
    string s;

    function append(byte c) {
        bytes(s).push(c);
    }

    function set(uint i, byte c) {
        bytes(s)[i] = c;
    }
}
```
### Can I concatenate two strings?
You have to do it manually for now.

## 未使用的gas会自动退款
gas的使用是交易的一部分，未使用的会立即自动退款。

## 当返回一个 `unit` 类型的值时，有没有可能返回 `undefined` 或像 `null` 类型之类的值？
这是不可能的，因为所有类型都占用了完整的数值范围。  
可以用 `throw` 处理有可能产生的错误或异常，它会恢复整个交易带来的所有改变，还原合约到交易前的状态。
如果你不想用 `throw` ，也可以返回一对参数帮助你处理异常，像这样：
```
pragma solidity ^0.4.0;

contract C {
    uint[] counters;

    function getCounter(uint index)
        returns (uint counter, bool error) {
            if (index >= counters.length)
                return (0, true);
            else
                return (counters[index], false);
    }

    function checkCounter(uint index) {
        var (counter, error) = getCounter(index);
        if (error) {
            // ...
        } else {
            // ...
        }
    }
}
```

## 部署合约时会包含注释吗？它是否会消耗gas?
不会，所有不执行的内容会在编译时被编译器删除，其中包括注释，变量名称和类型名称等。

## 如果在调用合约函数的交易中发送了Ether会怎么样？
Ether会被加入到合约的总余额中，但是函数必须有 `payable` 装饰器，否则会产生异常。

## 合约之间的交易会产生交易凭据吗？
不会，合约之间的函数调用不会创建自己的交易，必须查看整个交易。这也是为什么一般的区块链浏览器不能正确显示合约之间的Ether转移的原因。

## `memory` 关键字
Ethereum虚拟机有三个区域可以存储项目。第一个是存放合约所有状态变量的 `storage` ，每个合同都有自己的 `storage` ，它持久化存在且费用昂贵。第二个是“内存”，这是用来保存临时值。它在（外部）函数调用之间被擦除，并且使用起来更便宜。第二个是保存临时变量的 `memory` ，它在函数调用之间会被擦除，使用起来成本很低。第三个是堆栈，用于保存小的局部变量。它几乎可以免费使用，但只能保存有限数量的值。几乎所有的类型都无法指定存储位置，因为它们在每次使用时都会被复制。对存储位置重要的类型是结构体和数组，比如在函数调用中传递这些变量时，如果这些数据可以保留在 `memory` 或 `storage` 中，则不会复制它们的数据。这意味着可以在调用函数中的修改它们的内容，而且这些修改对于其他调用者仍然有效。默认存储位置取决于变量类型：状态变量始终处于 `storage` 中；函数参数总是在 `memory` 中；局部变量总是引用 `storage` 中的值。举例：
```
pragma solidity ^0.4.0;

contract C {
    uint[] data1;
    uint[] data2;

    function appendOne() {
        append(data1);
    }

    function appendTwo() {
        append(data2);
    }

    function append(uint[] storage d) internal {
        d.push(1);
    }
}
```
函数 `append` 可以作用于 `data1` 和 `data2` 两个变量，`data1` 和 `data2` 的改动会被永久保存。如果移除 `storage` 修饰符，会被看作函数参数默认存储在 `momery` 中，当调用 `append(data1)` 或 `append(data2)` 就会在 `momery` 中产生 `data1` 和 `data2` 两个状态变量的独立复制备份，并且 `append` 也有一个独立备份（不支持 `.push()` ，不过这是另一个问题）。这些独立备份的变动不会被写进 `data1` 和 `data2` 这两个状态变量。
一个常见的错误是声明一个局部变量，并假设它将在内存中创建，尽管它将在存储器中创建：
```
/// THIS CONTRACT CONTAINS AN ERROR

pragma solidity ^0.4.0;

contract C {
    uint someVariable;
    uint[] data;

    function f() {
        uint[] x;
        x.push(2);
        data = x;
    }
}
```
局部变量 `x` 的类型是 `uint[] storage x`，但由于 `storage` 不是动态分配的，所以必须从状态变量中分配，然后才能使用。因此，不会 `x` 分配存储空间，而是仅作为存储中预先存在的变量的别名。编译器会把它解释为指针，且默认指向 `storage` 中第 `0` 个位置，也就是 `someVariable` 存在的地方。所以 `someVariable` 会被 `x.push(2)` 修改。正确的写法如下：
```
pragma solidity ^0.4.0;

contract C {
    uint someVariable;
    uint[] data;

    function f() {
        uint[] x = data;
        x.push(2);
    }
}
```

## `bytes` 和 `byte[]` 有什么不同?
bytes is usually more efficient: When used as arguments to functions (i.e. in CALLDATA) or in memory, every single element of a byte[] is padded to 32 bytes which wastes 31 bytes per element.
 `bytes` 通常更有效：当用作函数的参数（即在CALLDATA）或 `momery` 中时，元素的每个 `byte[]` 被填充到32个字节，这样的话每个元素浪费了31个字节。

## Is it possible to send a value while calling an overloaded function?
It’s a known missing feature. https://www.pivotaltracker.com/story/show/92020468 as part of https://www.pivotaltracker.com/n/projects/1189488

Best solution currently see is to introduce a special case for gas and value and just re-check whether they are present at the point of overload resolution.
