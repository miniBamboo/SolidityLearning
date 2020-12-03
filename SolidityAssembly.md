Solidity汇编（Solidity Assembly)

solidity定义一个汇编语言，这个语言可以在没有Solidity下使用。也能在Solidity源代码中被用作“内联”。

内联汇编（Inline Assembly）

为了更细腻的控制，尤其是通过写库来提升语言，在一个接近虚拟即的语言中插入包含内联汇编的Solidity指令是完全可能的。因为EVM是一个堆栈机器，通常定位正确的堆栈槽以及在堆栈中正确的点提供参数给操作代码都是比较困难的。Solidity的内联汇编尝试使这些问题以及手动编写汇编时产生的其他争议简单化，通过下面的特征：

>>函数风格 的操作代码： mul(1,add(2,3)) 而不是 push1 3 push1 2 add push1 1 mul  
>>本地组件变量：let x := add(2,3) let y := mload(0x40) x := add(x,y)  
>>访问external变量：function f(uint x) public { assembly { x := sub(x,1) } }  
>>标签：let x := 10 repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))  
>>循环：for{ let i := 0 } lt(i, x) { i:= add(i, 1) } { y := mul(2,y) }  
>>if指令： if slt(x, 0) { x := sub(0, x) }  
>>switch指令： switch x case 0 { y := mul(x, 2) } default{ y := 0 }  
>>函数调用：
function f(x) -> y { switch x case 0 { y := 1 } default { y :+ mul(x, f(sub( x, 1))) } }  


操作代码（Opcodes）

如果一个操作代码使用了参数（总是从栈的顶端），它们会在圆括号中给出。注意参数的顺序在非函数风格的调用中可能反向（下面会解释）。用_标记的操作代码不会向栈中推入任何项目，而用×标记的操作码是特别的，而其他的所用操作码都会向栈中推入正好一个项目。标记有F，H，B或者C的操作码表示分别来自于Frontier, Homestead, Byzantium or Constantinople时期。其中Constantinople版本尚在规划中，使用任何标记有该信息的指令都会引起无效指令异常。

在下面的表格中，mem[a....b)表示内存（memory）字节从a位置开始直到位置b（但不包含位置b），storage[p]表示在p位置的存储（storage）内容。

操作码pushi 和 jumpdest无法被直接使用。

在语法（检查）中，操作码被表示为预定义的标识符。  

Instruction              Explanation  
stop - F stop execution, identical to return(0,0)   
add(x, y) F x + y   
sub(x, y) F x - y   
mul(x, y) F x * y   
div(x, y) F x / y   
sdiv(x, y) F x / y, for signed numbers in two’s complement   
mod(x, y) F x % y   
smod(x, y) F x % y, for signed numbers in two’s complement   
exp(x, y) F x to the power of y   
not(x) F ~x, every bit of x is negated   
lt(x, y) F 1 if x < y, 0 otherwise   
gt(x, y) F 1 if x > y, 0 otherwise   
slt(x, y) F 1 if x < y, 0 otherwise, for signed numbers in two’s complement   
sgt(x, y) F 1 if x > y, 0 otherwise, for signed numbers in two’s complement   
eq(x, y) F 1 if x == y, 0 otherwise   
iszero(x) F 1 if x == 0, 0 otherwise   
and(x, y) F bitwise and of x and y   
or(x, y) F bitwise or of x and y   
xor(x, y) F bitwise xor of x and y   
byte(n, x) F nth byte of x, where the most significant byte is the 0th byte   
shl(x, y) C logical shift left y by x bits   
shr(x, y) C logical shift right y by x bits   
sar(x, y) C arithmetic shift right y by x bits   
addmod(x, y, m) F (x + y) % m with arbitrary precision arithmetics   
mulmod(x, y, m) F (x * y) % m with arbitrary precision arithmetics   
signextend(i, x) F sign extend from (i*8+7)th bit counting from least significant   
keccak256(p, n) F keccak(mem[p…(p+n)))   
sha3(p, n) F keccak(mem[p…(p+n)))   
jump(label) - F jump to label / code position   
jumpi(label, cond) - F jump to label if cond is nonzero   
pc F current position in code   
pop(x) - F remove the element pushed by x      
dup1 … dup16 F copy ith stack slot to the top (counting from top)   
swap1 … swap16 * F swap topmost and ith stack slot below it   
mload(p) F mem[p..(p+32))   
mstore(p, v) - F mem[p..(p+32)) := v   
mstore8(p, v) - F mem[p] := v & 0xff (only modifies a single byte)   
sload(p) F storage[p]   
sstore(p, v) - F storage[p] := v   
msize F size of memory, i.e. largest accessed memory index   
gas F gas still available to execution   
address F address of the current contract / execution context   
balance(a) F wei balance at address a   
caller F call sender (excluding delegatecall)   
callvalue F wei sent together with the current call   
calldataload(p) F call data starting from position p (32 bytes)   
calldatasize F size of call data in bytes   
calldatacopy(t, f, s) - F copy s bytes from calldata at position f to mem at position t   
codesize F size of the code of the current contract / execution context   
codecopy(t, f, s) - F copy s bytes from code at position f to mem at position t   
extcodesize(a) F size of the code at address a   
extcodecopy(a, t, f, s) - F like codecopy(t, f, s) but take code at address a   
returndatasize B size of the last returndata   
returndatacopy(t, f, s) - B copy s bytes from returndata at position f to mem at position t   
create(v, p, s) F create new contract with code mem[p..(p+s)) and send v wei and return the new address   
create2(v, n, p, s) C create new contract with code mem[p..(p+s)) at address keccak256(   

. n . keccak256(mem[p..(p+s))) and send v wei and return the new address   
call(g, a, v, in, insize, out, outsize) F call contract at address a with input mem[in..(in+insize)) providing g gas and v wei and output area mem[out..(out+outsize)) returning 0 on error (eg. out of gas) and 1 on success   
callcode(g, a, v, in, insize, out, outsize) F identical to call but only use the code from a and stay in the context of the current contract otherwise   
delegatecall(g, a, in, insize, out, outsize) H identical to callcode but also keep caller and callvalue   
staticcall(g, a, in, insize, out, outsize) B identical to call(g, a, 0, in, insize, out, outsize) but do not allow state modifications   
return(p, s) - F end execution, return data mem[p..(p+s))   
revert(p, s) - B end execution, revert state changes, return data mem[p..(p+s))   
selfdestruct(a) - F end execution, destroy current contract and send funds to a   
invalid - F end execution with invalid instruction   
log0(p, s) - F log without topics and data mem[p..(p+s))   
log1(p, s, t1) - F log with topic t1 and data mem[p..(p+s))   
log2(p, s, t1, t2) - F log with topics t1, t2 and data mem[p..(p+s))   
log3(p, s, t1, t2, t3) - F log with topics t1, t2, t3 and data mem[p..(p+s))   
log4(p, s, t1, t2, t3, t4) - F log with topics t1, t2, t3, t4 and data mem[p..(p+s))   
origin F transaction sender   
gasprice F gas price of the transaction   
blockhash(b) F hash of block nr b - only for last 256 blocks excluding current   
coinbase F current mining beneficiary   
timestamp F timestamp of the current block in seconds since the epoch   
number F current block number   
difficulty F difficulty of the current block   
gaslimit F block gas limit of the current block   
