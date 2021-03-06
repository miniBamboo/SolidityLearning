# Solidity安全

算术运算安全可以使用通用的SafeMath 

## Solidity 未初始化存储指针安全风险  
要注意结构体，数组和映射的局部变量，在官方手册中有提到这些类型的局部变量默认是放在 storage 中的，因此这些局部变量可能都存在相同的问题。  
如下是问题代码，struct 在函数中被声明但是没有初始化，根据官方文档中可以知道，struct 在局部变量中默认是存放在 storage 中的，因此可以利用   
Unintialised Storage Pointers 的问题，p会被当成一个指针，并默认指向slot[0]和 slot[1] ，因此在进行p.name和 p.mappedAddress赋值的  
```solidity
pragma solidity ^0.4.11;
contract  testContract{

    bytes32 public testA; 
    address public testB;

    struct Person { 
        bytes32 name;  
        address mappedAddress;
    }

    function test(bytes32 _name, address _mappedAddress) public{
        Person p;
        p.name = _name;
        p.mappedAddress = _mappedAddress; 

    }
}
//同理数组也有同样的问题，如下是问题代码

contract C {
    uint public someVariable;
    uint[] data;

    function f() public {
        uint[] x;
        x.push(2);
        data = x;
    }
}
```  
解决方案
结构体 Unintialised Storage Pointers 问题的正确的解决方法是将声明的 struct 进行赋值初始化，  
通过创建一个新的临时 memory 结构体，然后将它拷贝到 storage 中。  
数组 Unintialised Storage Pointers 问题的正确解决方法是在声明局部变量 x 的时候，  
同时对 x 进行初始化操作。

``` solidity
pragma solidity ^0.4.11;
contract  testContract{

    bytes32 public testA; 
    address public testB;

    struct Person { 
        bytes32 name;  
        address mappedAddress;
    }

    mapping (uint => Person) persons;

    function test(uint _id, bytes32 _name, address _mappedAddress) public{
        Person storage p = persons[_id];
        p.name = _name;
        p.mappedAddress = _mappedAddress; 

    }
}


contract C {
    uint public someVariable;
    uint[] data;

    function f() public {
        uint[] x = data;
        x.push(2);
    }
}

```    
