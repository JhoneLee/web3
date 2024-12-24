# 全局属性与函数

## 1 数据结构

### 1.1 字节码

> 字节码是合约代码的编译后形式，用于在以太坊网络上部署一个新的智能合约，是智能合约的实际执行代码。部署后你才能通过合约的 ABI 与它交互。

用合约工厂部署智能合约 `byteCode` 就是 Solidity 编译后的字节码：

```javascript
const factory = ethers.ContractFactory.fromSolidity({
  abi,
  bytecode,
});
```

获取合约字节码：

```javascript
const code = await provider.getCode(address);
```

### 1.2 ABI

> ABI 有很多途径可以获取，并且有多种用途。

[ABI Documentation](https://docs.ethers.org/v6/api/abi/#Interface)

下面提到的 `encode` 函数都有对应的 `decode` 方法。`decode` 方法的第一个参数是接收一个 `Fragment` 对象，第二个参数是以 `0x` 开头的十六进制字符串数据。

#### 1.2.1 部署智能合约/实例化合约对象

此时的 ABI 可以通过 EtherScan 查询到合约的 ABI。如果使用 Hardhat，可以查看编译后的合约 JSON，获取 ABI 和字节码。

在 ethers.js 中获取合约的 ABI 可以通过 `interface` 属性生成 `Fragment`，再从 `Fragment` 对象转换为 ABI 字符串。

#### 1.2.2 调用合约方法

如果使用 ethers.js，不需要单独将参数序列化成 ABI 格式，直接给方法传入 JS 变量即可。

可以调用合约实例的 `interface` 对象的相关方法来编码和解码函数调用的参数。这个参数也是 `TransactionRequest` 接口中定义的 `data` 属性。

```javascript
const functionFragment = contract.interface.getFunction("transfer");
const values = ["0xaddress1", ethers.utils.parseUnits("100", 18)];
const data = contract.interface.encodeFunctionData(functionFragment, values);
```

#### 1.2.3 编码合约事件日志

编码事件日志的内容，生成事件的 `data` 和 `topics` 部分：

```javascript
// 获取事件描述
const eventFragment = contract.interface.getEvent("Transfer");

// 假设我们要模拟一个 Transfer 事件的日志，参数包括 from 地址，to 地址，以及 tokenId
const values = [
  "0xaddress1", // from
  "0xaddress2", // to
  123, // tokenId
];

// 编码事件日志
const { data, topics } = contract.interface.encodeEventLog(
  eventFragment,
  values
);
```

#### 1.2.4 编码事件过滤器

```javascript
const eventFragment = contract.interface.getEvent("Transfer");
const values = ["0xaddress1", "0xaddress2"];
const topics = contract.interface.encodeFilterTopics(eventFragment, values);
```

`values` 表示两个地址，`from` 和 `to`。

#### 1.2.5 编码合约函数执行结果

```javascript
const functionFragment = contract.interface.getFunction("balanceOf");
const values = [ethers.utils.getAddress("0xaddress1")];
const resultData = contract.interface.encodeFunctionResult(
  functionFragment,
  values
);
```

- 获取合约的接口（Contract Interface）：每个合约都有一个 ABI，它定义了函数、事件、构造函数等的输入输出格式。
- 编码和解码交易数据时：ABI 用于将 JS 对象（或其他格式的数据）转换为以太坊虚拟机（EVM）能够理解的字节流，以便进行交易。

### 1.3 Fragment

> 将 ABI 字符串封装成为对象。

类型：

- 合约方法 Fragment
- 事件 Fragment
- 合约构造函数 Fragment

通过合约对象的 `interface` 属性的 `getFunction` 和 `getEvent` 获取，转换为 ABI：

```javascript
const transferFragment = iface.getFunction("transfer");

// 获取 ABI 字符串表示
const abiString = transferFragment.format(); // 也可以是 formatJson
```

### 1.4 交易类型

> 用整数表示的交易类型：

- **Type 0**: 传统交易类型，通常用于发送 ETH 或调用合约。
- **Type 1**: EIP-2930 引入了 `accessList` 提升链上存储访问效率。
- **Type 2**: EIP-1559 交易类型，改进了 Gas 费用机制，让 Gas 费分为基础费 + 优先费。
- **Type 3**: EIP-4844 交易类型，提供了对 Blob 数据结构的支持，这主要是为了支持更高效的大量数据传输，适用于未来的分片和数据存储等优化。

### 1.5 链

- **chainId**: 每个链都有自己独特的 ID，比如以太坊主网和 Sepolia 测试网就不同。

### 1.6 区块

- **blockTag**: 每个区块都有自己的 hash 值。

### 1.7 其他

- **EtherSymbol**：以太坊符号的 Unicode 编码
- **MaxInt256**：最大有符号整数
- **MaxUint256**：最大无符号整数
- **MessagePrefix**：标准消息前缀
- **MinInt256**：最小有符号整数
- **N**：表示 secp256k1 曲线基点的阶（order）N
- **WeiPerEther**：1 Ether 多少个 Wei 的换算
- **ZeroAddress**：全 0 地址，表示无效地址
- **ZeroHash**：全 0 哈希，表示无效哈希

## 2 全局函数

- **getAddress**: `(address: string) => string`  
  用来返回一个规范化（normalized）且经过校验和（checksum）的以太坊地址。它的作用是确保你提供的地址符合以太坊地址标准，并且在有需要时会进行校验和验证或生成。地址的标准是 **40 个十六进制字符**。

  **校验和**是通过地址的哈希值来生成的。以太坊将地址的哈希（通过 keccak256 算法）进行处理，并将哈希中的某些位映射到地址中的字符大小写。具体来说，地址中的字符位置如果对应哈希值的某些位是 1，则该字符为大写，如果是 0，则为小写。这种方法能在一定程度上防止地址输入错误。

  - 如果传入的地址没有大小写区分（比如全小写或全大写），`getAddress` 会将其转换成标准的校验和地址，并确保大小写正确。
  - 如果地址已经符合校验和规则，`getAddress` 会验证其是否符合要求。如果地址的校验和无效，函数会抛出错误。
  - 如果你不关心地址的校验和，想要忽略它，你可以将地址转换为全小写，然后传递给 `getAddress`。这将绕过校验和检查（虽然这种情况很少见）。

### 2.1 copyRequest

> 输入一个 TransactionRequest 对象， 返回一个深拷贝的新对象

我们需要这么做，是为了能复用交易对象，还不污染原来的；

### 2.2 getDefaultProvider

#### 参数

- network: string 如果传入 websocke url 返回一个 WebSocketProvider, 如果传入的 http url 返回一个 JsonRpcProvider
- options: 用来定制网络提供者的行为, 看你前面的传入的 network 参数需要啥就传啥
  目前支持的 network 有：
  - alchemy
  - ankr
  - cloudflare
  - chainstack
  - etherscan
  - infura
  - publicPolygon
  - quicknode
