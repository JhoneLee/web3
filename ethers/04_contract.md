# 合约对象

[文档](https://docs.ethers.org/v6/api/contract/)

## 1 合约对象的创建

需要智能合约的 abi 地址， 如果只读需要 provider， 如果读写需要 signer

```javascript
abi = [
  "function decimals() view returns (string)",
  "function symbol() view returns (string)",
  "function balanceOf(address addr) view returns (uint)",
];

// Create a contract
contract = new Contract(contractAddress, abi, provider);
```

## 2 合约基类 BaseContract

> 合约类 Contract 也继承自 BaseContract

### 2.1 固定属性

- fallback null|WrappedFallback 就是智能合约的 fallback 那个概念，fallback 是智能合约中的一种转账记录的钩子函数
  ```javascript
  contract.fallback;
  ```
- filters: 合约中提供的所有事件, 具体使用时作为日志筛选器的参数用，直接调用没用。 假设我的智能合约有 logReceive 事件, 需要和 queryFilter 方法配合
  ```javascript
  contract.filters.logReceive();
  ```
- interface: 合约接口 和 `ContractFactory`的`interface`一样
- runner: 合约的执行者，实现了 ContractRunnber 接口，提供了用于调用合约函数的方法
- target: 合约初始的地址， 可以是地址或 ENS 域名(有的合约 abi 相同，但是地址不一样)

### 2.2 方法

#### 事件及日志

- addListener/on : 合约事件监听
- emit: 触发一个合约事件 (event: ContractEventName, args: Array< any >)⇒ Promise< boolean >
- off: 解绑一个事件监听
- once: 绑定只触发一次的 handler
- queryFilter 查询和过滤指定事件的历史日志。它允许你按区块范围筛选并获取某个事件的日志数据。所谓区块范围就是区块的高度值，第一个参数是事件名称，第二个参数是起始区块 第三个参数是结束区块，默认查询 0-latest
- removeAllListeners 解除某个事件的全部绑定
- removeListener 结仇某事件的某个绑定

#### 创建新合约

- attach: 传入不同的合约地址，在当前合约基础上（runner 不变 abi 不变）生成一个新合约对象
- connect: 传入一个新的 provider 或者 signer， 在当前合约基础上(abi 不变 target 不变) 生成一个新合约对象

#### 合约部署

- deploymentTransaction 当用工厂合约部署合约的时候，会返回一个 合约部署交易对象
- waitForDeployment 等待合约部署完毕

#### 合约元数据

- getAddress 获取当前合约实例对应的合约地址
- getDeployedCode 获取当前合约实例对应合约的字节码
- getFunction 获取当前合约的一个函数体，参数是函数名称，用于在合约函数命名和 js 关键字冲突的情况，比如智能合约提供了一个 prototype 函数，在 js 里面 合约实例.prototype 显然是执行不了的, 同时也可以获取一个 BaseContractMethod 实例
- listenerCount： 输入事件名，返回这个事件的监听者的数量
- listeners：输入事件名，返回这个事件的全部监听者数组

#### 静态方法

- buildClass 通过传入合约 abi, 生成一个新的合约类，供 new 出合约实例使用, 这么搞的优点在于：

  - 对某些结构相同的合约可以单独命名，让代码更具有语义化，
  - 同时在合约 ABI 不确定或可能更新的情况下，可以封装合约类工厂，批量生成合约类

  ```javascript
  // 创建 ERC-20 合约类
  const ERC20 = ethers.BaseContract.buildClass(abi);

  // 实例化不同的代币合约
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");

  const token1 = new ERC20("0xTokenAddress1", provider);
  const token2 = new ERC20("0xTokenAddress2", provider);
  ```

- from 通过指定的合约地址（target）、ABI 和运行器（runner），创建一个新的 BaseContract 实例，类似 new Contract(address, abi, runner) 的替代方法，但更加底层
  ```javascript
  const contract = ethers.BaseContract.from(contractAddress, abi, provider);
  ```

## 3 BaseContractMethod 接口

> 为**合约函数**（注意不是合约）提供一个 抽象层，描述这些方法的行为和交互方式。它的设计目的是为了在 TypeScript 环境下增强类型安全性，同时方便开发者管理和调用合约方法。

这个接口主要用于描述合约的方法，通过 Contract 实例的 getFunction(函数名称) 可以获取这个方法

```javascript
contract.getFunction("getUserName");
```

### 3.1 属性

- fragment： 表示当前智能合约函数 ABI 的具体定义

```typescript
FunctionFragment {
    name: "transfer",
    type: "function",
    inputs: [
        { name: "to", type: "address", indexed: false, components: null },
        { name: "amount", type: "uint256", indexed: false, components: null }
    ],
    outputs: [
        { name: "", type: "bool", components: null }
    ],
    stateMutability: "nonpayable",
    constant: false,
    payable: false
}
```

- name: 合约件数的名称

### 3.2 方法

- estimateGas： gas 费评估 参数是这个合约函数需要传递的参数， 返回值是 wei 单位的 bigint
- getFragment: 参数是这个合约函数的参数，返回带参数的 ABI 片段
- populateTransaction： 用于生成一个完整的交易请求对象（TransactionRequest)。这个功能非常适合在交易发送前检查交易内容，或者将交易对象传递给其他工具或服务（例如多签钱包或离线签名工具）
  ```javascript
  await contract.getFunction("getUserName").populateTransaction();
  // 带参数： 前面的都是智能合约函数需要的参数，最后一个是配置项，可以设置 gas 上限等等
  const txReq2 = await contract
    .getFunction("setInfo")
    .populateTransaction(98n, "jack", true, {
      gasLimit: 100000,
    });
  ```
- send: 执行一笔会修改区块链状态的交易，例如发送 ETH 或调用非 view/pure 的智能合约方法
- staticCall: 是模拟执行，不会发送真实交易，也不会消耗 Gas
- staticCallResult: 和 staticCall 的效果一样，只不过返回值不同
  ```javascript
  // 如果是单一返回值 staticCall 返回的是提炼后的值
  const tx3 = await contract.getUserName.staticCall(); // jack
  const tx4 = await contract.getUserName.staticCallResult(); // {0:''}
  // 如果是多个返回值 return(a,b,v)，那二者返回是一样的 {0:a,1:b,2:c}
  ```

## 4 ContractFactory 类

> 被用于在 hardhat 等开发框架中，部署合约

经典部署脚本, 这个脚本是在 Nodejs 中运行的

```javascript
const { ethers } = require("hardhat");

async function main() {
  const Counter = await ethers.getContractFactory("Counter");
  const counter = await Counter.deploy();
  await counter.waitForDeployment();
  console.log("Counter address:", await counter.getAddress());
}

main();
```

### 4.1 属性

- bytecode: 被部署合约的字节码
- interface: 被部署合约的 Interface 对象
  - 编码函数调用数据：interface.encodeFunctionData 方法可以将人类可读的合约函数调用转换成 EVM 可执行的字节码数据
  - 提供对合约内函数、事件的完整描述，这在动态生成交互逻辑或调试时非常有用, 例如 getFunction getEvent 等, 返回 Fragment 对象，描述了它们的结构
  - 解码事件日志 decodeEventLog 将日志字节码数据转换成人类可读数据
  - 过滤事件日志 encodeEventTopic 通过传入某事件的 eventFragment 进行过滤
  - 生成 abi json 数据， ContractX.interface.formatJson()
- runner: 合约工厂的执行者

## 4.2 构造函数

入参

- abi: 必填 合约的 abi
- bytecode: 必填 合约的字节码
- runner：选填 执行者

如果是 hardhat 的 ethers.ContractFactory, 只需要填入一个合约名称即可

## 4.3 方法

- attach: 道理和 BaseContract 差不多，但是目标是 ContractFactory 所产出的那个合约对象
- connect 同理
- deploy：部署合约，参数是合约的 constructor 参数
- getDeployTransaction：获取当前部署智能合约这笔交易的 Transaction 对象

## 4.4 静态方法

- fromSolidity： 通过智能合约的 abi 和 字节码创建该合约的 ContractFactory 实例，部署智能合约
  ```javascript
  const factory = ethers.ContractFactory.fromSolidity({
    abi,
    bytecode: bytesCode,
  });
  const deployFactory = factory.connect(wallet);
  const contract = await deployFactory.deploy();
  ```

## 5 ContractTransaction 接口

三个属性

- data: 交易数据信息
- from: 交易 Maker 地址
- to: 交易 Takerd 地址

## 6 EventLog 类

### 6.1 属性

- args: 经过解析后的事件传参， 由智能合约调 emit 方法传入的数据
- eventName: 事件名称
- eventSignature: 事件签名
- fragment: 事件的 fragment 对象， 事件结构
- interface: 事件的 interface 接口

## 7 WrappedFallback 接口

对合约的转账回退函数的额外属性的修饰

### 7.1 方法

- estimateGas: `(overrides?: Omit< TransactionRequest, "to" >)⇒ Promise< bigint >` 预估 fallback 的 gas 消耗
  - 如果 overfides 里面写了 gasLimit 并且 gasLimit 比实际返回的数据小， 会抛出 Error: missing revert data 错误
  - 对于非接收型的 fallback, data 参数传递了有可能会被覆盖
  - 传递 data 一定要用 byteslike 的格式 `const data = ethers.toUtf8Bytes('mock ok')`;
  ```javascript
  // 不能传 undefined, 要传递一个空对象 或者实际数据
  await payContract.fallback.estimateGas({
    data,
    from: wallet.address,
    gasLimit: 100000,
  });
  ```
- populateTransaction： `(overrides?: Omit< TransactionRequest, "to" >)⇒ Promise< ContractTransaction >` 返回 fallback 的交易请求对象， 同样的，如果是非接收型 fallback,data 会被覆盖
- send： `(overrides?: Omit< TransactionRequest, "to" >)⇒ Promise< ContractTransactionResponse > `发送一个交易个 fallback 函数
- staticCall： `(overrides?: Omit< TransactionRequest, "to" >)⇒ Promise< string > `调用 fallback 函数并返回一个结果，同样的，如果是非接收型 fallback,data 会被覆盖

## 8 ContractRunner 接口

> 定义了能够和合约交互的对象

下面的 contract 实例的 runner 就是 ContractRunner

```javascript
const provider = new ethers.JsonRpcProvider("https://your-ethereum-node");
const contract = new ethers.Contract(contractAddress, contractABI, provider);
const { runner } = contract;
```

### 8.1 方法

- call: `(tx: TransactionRequest) => Promise< string >` 方法通常用于调用合约中的 "view" 或 "pure" 函数, 返回的结果是 abi 编码的十六进制字符串，想要用需要额外解析
  ```javascript
  const result = await contract.runner.call(txReq);
  const abiCoder = new ethers.AbiCoder();
  abiCoder.decode(["string"], result);
  ```
- estimateGas：`(tx: TransactionRequest) => Promise<bigint>` 预估消耗的 gas 个数，返回值是 bigint. 如果交易设置 gasLimit, 并且比估算出来的值低，这个方法会抛出错误
- provider: 获取当前 runner 的查询者
- resolveName: 获取一个 ENS 域名的地址， 前提是当前链支持 ENS 域名
- sendTransaction：发送一笔交易 `(tx: TransactionRequest) => Promise< TransactionResponse >`
