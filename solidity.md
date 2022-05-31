immutable constructor 中 设置 不能改变
constant 变量可以节省gas
类全局是区块链变量, 改变区块链变量 需要发送交易, 读取不用

Car car = (new Car){value: msg.value, salt: _salt}(_owner, _model);
Car car = new Car(_owner, _model);

first 4 bytes as selecttor
addr.call(abi.encodeWithSignature("transfer(address,uint256)", 0xSomeAddress, 123))
    function getSelector(string calldata _func) external pure returns (bytes4) {
        return bytes4(keccak256(bytes(_func)));
    }
}

  (bool sent, ) = _to.call{value: msg.value}("");

(bool success, bytes memory data) = _addr.call{value: msg.value, gas: 5000}(
    abi.encodeWithSignature("foo(string,uint256)", "call foo", 123)
);


receive() external payable
fallback() external payable

```sol
try / catch can only catch errors from external function calls and contract creation.

try foo.myFunc(_i) returns (string memory result) {
    emit Log(result);
} catch {
    emit Log("external call failed");
}
```

try func(param) return (){
    
}catch{

}

library Array {
    function remove(uint[] storage arr, uint index) public {
    }
}



library
    using Array for uint[];

    uint[] public arr;
    arr.remove(1);


节省gas
- Replacing memory with calldata
- Loading state variable to memory
- Replace for loop i++ with ++i
- Caching array elements
- Short circuit


```
    // gas optimized
    // [1, 2, 3, 4, 5, 100]
    function sumIfEvenAndLessThan99(uint[] calldata nums) external {
        uint _total = total;
        uint len = nums.length;

        for (uint i = 0; i < len; ++i) {
            uint num = nums[i];
            if (num % 2 == 0 && num < 99) {
                _total += num;
            }
        }

        total = _total;
    }
```


owner != address(0)
require(!isOwner[owner], "owner not unique");
(bool success, ) = transaction.to.call{value: transaction.value}(
    transaction.data
);



signature / public_secret = ethSignedMessageHash 签名和验证都要做这一步

ethSignedMessageHash * private_secret = signature

signature / ethSignedMessageHash = pulic_secret



在Solidity中constant、view、pure三个函数修饰词的作用是告诉编译器，函数不改变/不读取状态变量，这样函数执行就可以不消耗gas了（是完全不消耗！），因为不需要矿工来验证，所以用好这几个关键词很重要！




用户需要确保initialize函数只被调用一次。我们这样做，是通过检查所有者是否是地址(0)。


erc1155协议 发行管理代币


byte4 sig = byte4(keccak256("bFunction(uint256,string)"));
bytes memory _bnum =  abi.encode(_num);
bytes memory _bMessage  = abi.encode(_message);
(bool , succcess) = address.call(
    abi.encodePacked(sig,_bNum,_bMessage)
)

(bool , succcess) = address.call(
    abi.encodeWithSignature("bFunction(uint256,string)",_num,_message)
)


)

byte4 sig = byte4(keccak256("bFunction(uint256,string)"));

(bool , succcess) = address.call(
    abi.encodeWithSelector(sig,_num,_message)
)


fallabck函数是solidity里面实现的一个无名函数。
必须加 external payable
    V6  版本solidity 的fallback 实现。

     fallback() external payable {x = 1;k=1; y = msg.value; }
    receive() external payable { 
        x = 2; 
        y = msg.value; 
        
    }



问：Geth客户端中都有哪些API（Application Programming Interface，应用程序编程接口）？
答：Admin（管理员）、 eth（以太币）、web3、miner（矿工）、net（网络）、personal（个人）、shh、debug（调试）和 txpool（工具）。


问：如何将自定义javascript文件加载到Geth控制台？
答：输入”—preload”命令和文件的路径即可。

问：Geth客户端是否能用来挖矿？
答：是的，输入“—mine”命令即可。


solidity. msg.data