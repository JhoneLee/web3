# Public Actions

> 它们是与以太坊 RPC 方法（如 eth_blockNumber、eth_getBalance 等）一一对应的操作，通常用在与公共客户端进行交互时。

**用途与特点**

- 主要用于读取以太坊区块链上的公开信息， 不提供签名功能，只读
- 涉及敏感的签名操作，所以它们是相对安全的。用户不需要提供私钥或进行任何身份验证。

[文档地址](https://viem.sh/docs/actions/public/introduction)

**主要涵盖**

- 账户信息查询
  - [getBalance](https://viem.sh/docs/actions/public/getBalance) 账户余额
  - [getTransactionCount](https://viem.sh/docs/actions/public/getTransactionCount) 某地址的交易总和
- 区块信息查询
  - [getBlock](https://viem.sh/docs/actions/public/getBlock) 获取区块信息
  - [getBlockNumber](https://viem.sh/docs/actions/public/getBlockNumber) 获取最新的区块高度
  - [getBlockTransactionCount](https://viem.sh/docs/actions/public/getBlockTransactionCount)区块交易数量
  - [watchBlockNumber](https://viem.sh/docs/actions/public/watchBlockNumber)监听最新的区块高度变化
  - [watchBlocks](https://viem.sh/docs/actions/public/watchBlocks) 持续监听最新打包的区块信息
- [call](https://viem.sh/docs/actions/public/call) (和 ethers providr.call 一个性质)
- 链信息
  - [getChainId](https://viem.sh/docs/actions/public/getChainId) 获取链 id
- EIP-712
  - [getEip712Domain](https://viem.sh/docs/actions/public/getEip712Domain) 用于获取符合 EIP-712 规范合约的 domain 信息
- 费用相关
  - [estimateFeesPerGas](https://viem.sh/docs/actions/public/estimateFeesPerGas) 估算当前情况下每个 gas 费用，可选择估算 EIP-1559 也可估算原始的交易（不是吧交易当参数传，是计算一个通用的）， 可设置链 ID。
  - [estimateGas](https://viem.sh/docs/actions/public/estimateGas) 估算一个交易的 gas 消耗
  - [estimateMaxPriorityFeePerGas](https://viem.sh/docs/actions/public/estimateMaxPriorityFeePerGas) 估算优先费
  - [getBlobBaseFee](https://viem.sh/docs/actions/public/getBlobBaseFee) 获取 blob 数据存储的基础费
  - [getFeeHistory](https://viem.sh/docs/actions/public/getFeeHistory) 获取历史费用信息
  - [getGasPrice](https://viem.sh/docs/actions/public/getGasPrice) 获取原始交易的每 gas 费价格
- 日志及筛选器
  - [createBlockFilter](https://viem.sh/docs/actions/public/createBlockFilter) 创建一个区块筛选器，用于监听和跟踪区块链状态的变化，对于区块过滤器，这意味着可以查看自创建过滤器以来的所有新区块哈希
  - [createEventFilter](https://viem.sh/docs/actions/public/createEventFilter) 创建事件筛选器
  - [createPendingTransactionFilter](https://viem.sh/docs/actions/public/createPendingTransactionFilter) 获取等待中的交易过滤器
  - [getFilterChanges](https://viem.sh/docs/actions/public/getFilterChanges) 通过传入上面的 filter 对象，可以获取**上一次调用到当前调用时间内** 对应的日志。日志返回的内容都是哈希编码，需要额外解析后才能看
  - [getFilterLogs](https://viem.sh/docs/actions/public/getFilterLogs) 获取事件 filter 的日志
  - [getLogs](https://viem.sh/docs/actions/public/getLogs) 获取全部事件日志
  - [watchEvent](https://viem.sh/docs/actions/public/watchEvent) 观察链上事件日志，可以根据 地址、事件名等筛选
  - [uninstallFilter](https://viem.sh/docs/actions/public/uninstallFilter) 销毁传入的 filter
- 默克尔证明(是用来验证某个数据项（如交易、账户状态等）是否包含在一个 Merkle 树中的一种技术)
  - [getProof](https://viem.sh/docs/actions/public/getProof) 获取一个智能合约插槽的默克尔证明
- 签名校验
  - [verifyMessage](https://viem.sh/docs/actions/public/verifyMessage) 验证消息原文、地址和签名后信息是否相同， 返回 bool
  - [verifyTypedData](https://viem.sh/docs/actions/public/verifyTypedData) 验证满足 EIP-712 的加签数据 typed 数据部分是否和签名一致
- 交易数据
  - [prepareTransactionRequest](https://viem.sh/docs/actions/wallet/prepareTransactionRequest) 在给出基本的 account value to 之后装填剩余的 txReq 数据信息
  - [getTransaction](https://viem.sh/docs/actions/public/getTransaction) 根据交易 hash 获取交易信息
  - [getTransactionConfirmations](https://viem.sh/docs/actions/public/getTransactionConfirmations) 计算传入的交易回执对象或者交易哈希 的确认数，确认数不达标表示这笔交易存在作废的可能
  - [getTransactionReceipt](https://viem.sh/docs/actions/public/getTransactionReceipt) 根据传入的交易 hash 获取这笔交易的交易回执对象
  - [sendRawTransaction](https://viem.sh/docs/actions/wallet/sendRawTransaction) 发送一笔交易，这个交易用十六进制编码的方式传入，所以是 RawTransaction, public client 也可以调，因为你单独查询也算是交易
  - [waitForTransactionReceipt](https://viem.sh/docs/actions/public/waitForTransactionReceipt) 传入交易的 hash，等待这笔交易返回的交易回执对象， 可以设定确认数，确认数达标后再返回这笔交易的交易回执
  - [watchPendingTransactions](https://viem.sh/docs/actions/public/watchPendingTransactions) 观察处于等待中的交易

## 1 账户操作

### 1.1 getBalance

对应的 JSON RPC 方法： eth_getBalance

[接口文档](https://viem.sh/docs/actions/public/getBalance)

使用方式
javascript
import { publicClient } from './client'

const balance = await publicClient.getBalance({
address: '0xA0Cf798816D4b9b9866b5330EEa46a18382f251e',
})

// 10000000000000000000000n (wei)

**参数**

- address: 地址字符串
- blockNumber： bigint 区块高度
- blockTag： 区块标签 'latest' | 'earliest' | 'pending' | 'safe' | 'finalized'

**返回值**： bigint wei 单位

### 1.2 getTransactionCount

返回一个地址广播出去或者接收到的交易数量

对应 JSON RPC eth_getTransactionCount

[接口文档](https://viem.sh/docs/actions/public/getTransactionCount)

使用方式
javascript
import { publicClient } from './client'

const transactionCount = await publicClient.getTransactionCount({  
 address: '0xA0Cf798816D4b9b9866b5330EEa46a18382f251e',
})
// 420

**参数**

- address: 地址
- blockNumber: bigint 区块高度
- blockTag： 'latest' | 'earliest' | 'pending' | 'safe' | 'finalized' 区块标签

## 2 call

> 和 ethers.js 的 provider.call 有异曲同工之妙， 调用合约修改状态方法并不会发起一个真实的区块链交易，而是进行本地模拟

对应 JSON RPC 函数 eth_call

**主要功能：**

- 无 gas 消耗调用智能合约 pure view 函数
- 调用未部署合约的函数
- 模拟调用合约的 gas 消费

返回值是经过编码的十六进制字符串，把智能合约的执行结果原样返回了。

### 2.1 使用方式

**调用 pure view 函数**
javascript
import { account, publicClient } from './config'

const data = await publicClient.call({
account,
data: '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2',
to: '0x70997970c51812dc3a010c7d01b50e0d17dc79c8',
})

**调用未部署合约函数**
通过 byteCode 的方式
javascript
mport { encodeFunctionData, parseAbi } from 'viem'
import { publicClient } from './config'

const data = await publicClient.call({
// Bytecode of the contract. Accessible here: https://etherscan.io/address/0xFBA3912Ca04dd458c843e2EE08967fC04f3579c2#code
code: '0x...',
// Function to call on the contract.
data: encodeFunctionData({
abi: parseAbi(['function name() view returns (string)']),
functionName: 'name'
}),
})

通过合约工厂的方式
javascript
import { encodeFunctionData, parseAbi } from 'viem'
import { owner, publicClient } from './config'

const data = await publicClient.call({
// Address of the contract deployer (e.g. Smart Account Factory).
factory: '0xE8Df82fA4E10e6A12a9Dab552bceA2acd26De9bb',

// Function to execute on the factory to deploy the contract.
factoryData: encodeFunctionData({
abi: parseAbi(['function createAccount(address owner, uint256 salt)']),
functionName: 'createAccount',
args: [owner, 0n],
}),

// Function to call on the contract (e.g. Smart Account contract).
data: encodeFunctionData({
abi: parseAbi(['function entryPoint() view returns (address)']),
functionName: 'entryPoint'
}),

// Address of the contract. 这里存在疑问，to 究竟是什么
to: '0x70997970c51812dc3a010c7d01b50e0d17dc79c8',
})

### 2.2 参数

- account: 相当于 from 参数，调用发起方的地址
- data: 经过编码后的函数调用哈希值，具体怎么生成看上面的例子
- to: 调用的目标合约地址 或者 接收者
- accessList： 智能合约存储槽位置 数据结构 {address: '可能访问的合约地址', storageKey:'存储槽位置'}
- blockNumber： 区块高度
- blockTag ： 区块标签 'latest' | 'earliest' | 'pending' | 'safe' | 'finalized' 默认 latest
- code: call 要调用的字节码
- factory： 工厂合约地址
- factoryData： 工厂合约部署函数及传参的哈希字符串
- gas: 模拟本次交易消耗的 gas，如果设置的不足会导致模拟报错，用于预估本次交易的 gas 消耗
- gasPrice: 模拟本次交易每个 gas 的价格，只能应用于最原始版本的交易
- maxFeePerGas： 每 gas 最大费用，用于 EIP-1559
- maxPriorityFeePerGas: 每 gas 最大优先费用，用于 EIP-1559
- nonce: 模拟交易单号，不设置会报错
- stateOverride： 在调用某个合约函数时，临时修改或覆盖特定合约或账户的状态，而这些改变只会在当前函数调用过程中生效。调用结束后，状态会恢复到原来的样子。它不会真正地提交到区块链上，只是模拟状态变化。
- value: 模拟转账金额

## 3 getEip712Domain

> IP-712 是一种规范，用于 结构化数据的签名和验证，特别是针对以太坊智能合约中的数据签名，它定义了一种标准的域（domain）结构，确保签名数据的一致性和安全性。该规范确保了用户签名的数据是符合预定格式的，从而增强了数据的安全性和可验证性。

getEip712Domain 是一个用于读取合约中 EIP-712 domain 的方法，它会根据 ERC-5267 规范，返回合约的域信息。具体来说，它会返回以下内容：

- domain：表示 EIP-712 domain 的基本信息。
- extensions：与 EIP-712 domain 相关的扩展信息，可以包含更多的自定义数据。
- fields：描述 EIP-712 domain 的字段，通常是一些关于数据结构的定义，比如合约名称、版本、链 ID 等。

### 3.1 使用方式

javascript
import { publicClient } from './client'

const { domain, extensions, fields } = await publicClient.getEip712Domain({
address: '0x57ba3ec8df619d4d243ce439551cce713bb17411',
})

### 3.2 模拟调用

合约尚未部署的情况下，模拟获取该合约的 domain 信息。 通过合约工厂实现
javascript
import { factory, publicClient } from './config'

const { domain, extensions, fields } = await publicClient.getEip712Domain({
address: '0x57ba3ec8df619d4d243ce439551cce713bb17411',
factory: factory.address,
factoryData: encodeFunctionData({
abi: factory.abi,
functionName: 'createAccount',
args: ['0x0000000000000000000000000000000000000000', 0n]
}),
})

### 3.3 参数

- address：满足 EIP-712 草案的合约地址
- factory：合约工厂地址
- factoryData：合约工厂的传输数据
