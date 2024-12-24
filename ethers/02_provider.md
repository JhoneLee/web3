# Provider 相关

## 1 Provider 接口

> 继承自 ContractRunner, EventEmitterable, NameResolver , 主要负责获取以太坊上只读信息的获取。 涵盖 **账户、区块、交易、事件日志及模拟智能合约执行**

- 账户数据：
  - 账户余额：你可以通过 provider.getBalance(address) 获取某个地址的以太坊余额。
  - 交易计数（Nonce）：通过 provider.getTransactionCount(address) 获取账户的交易计数，也就是账户已发送交易的数量。
  - 账户代码：通过 provider.getCode(address) 获取账户的代码（如果是合约地址的话），这个方法可以让你检查合约地址是否存在。
  - 状态存储：你可以通过 provider.getStorageAt(address, position) 来查询某个地址的存储内容（比如合约的变量值）。
- 区块信息
  - 区块信息：provider.getBlock(blockNumber) 用来获取某个区块的详细信息。
  - 区块内交易：provider.getBlockWithTransactions(blockNumber) 会返回区块中的交易信息。
  - 最新区块：通过 provider.getBlockNumber() 可以获取当前最新区块的编号。

初始化分为三种情况，一种是用户安装了 MetaMask, 可以派生出 signer 对象

```js
provider = new ethers.BrowserProvider(window.ethereum);
```

另一种用户没装，给一个默认的, 默认的无法提供 signer

```javascript
provider = ethers.getDefaultProvider();
```

还有一种是连接第三方的 JsonRpc 服务，可以派生 signer

```javascript
provider = new ethers.JsonRpcProvider("YOUR_INFURA_URL");
```

### 1.1 属性

- provider： this 就是自己

### 1.2 方法

#### 账户相关

- **getBalance**： (address: AddressLike, blockTag?: BlockTag)⇒ Promise< bigint > 获取某个账户地址的余额， 如果传入 blockTag, 那么余额就是指当前账户在这个区块打包时他的余额是多少
- **getCode** : (address: AddressLike, blockTag?: BlockTag)⇒ Promise< string > 获取当前地址的字节码， 如果是普通用户的地址，返回空字节码 0x 如果第二个 BlockTag 传入了非部署此合约的区块，会报错 Error: could not coalesce error

#### 交易相关

- **broadcastTransaction** ：(signedTx: string)⇒ Promise< TransactionResponse > 将一个已经通过 Signer 签名过的交易提交到以太坊网络的内存池（mempool）中，等待矿工打包成区块, 相当于执行了交易。这个 singnedTx 要求比较苛刻

```javascript
// 先获取一个只和交易有关的交易请求对象
const tx = await payContract
  .getFunction("pay")
  .populateTransaction(0n, { value: amount });
// 补充必要的缺失信息
const nonceManager = new ethers.NonceManager(wallet);
tx.nonce = await nonceManager.getNonce();
tx.maxFeePerGas = ethers.parseEther("0.0001");
tx.gasLimit = 50 * 1000;
tx.chainId = (await provider.getNetwork()).chainId;
const signedTx = await wallet.signTransaction(tx);
const txRep = await provider.broadcastTransaction(signedTx);
```

- **call** : (tx: TransactionRequest)⇒ Promise< string > 预执行这笔交易， 并不会发起真的合约请求；如果预执行失败会触发 CallExceptionError 错误； 只有在 call view 或 pure 有返回值函数的时候，才能取到十六进制编码的返回值。 不满足上述条件没有返回值
- **destory** : void 关闭并清理与 Provider 相关的资源， 防止后续对该 Provider 的调用，确保资源被释放， 如果再用这个 provider 对象会报错；
- **estimateGas**: (tx: TransactionRequest)⇒ Promise< bigint > 获取这个交易的预估 gas 消耗数量
- **getTransaction** (hash: string)⇒ Promise< null | TransactionResponse > 根据 TransactionResponse 的 hash 字段的取值 获取一个 TransactionResponse 对象，如果这个交易没有打包到区块或者区块在链分叉中被丢弃，返回 nulll
- **getTransactionReceipt** ： (hash: string)⇒ Promise< null | TransactionReceipt > 和上一个函数原理一样，只不过返回的是交易回执 这里的 hash 无论是 TransactionRequest 还是 TransactionReceipt 都是同一个。
- **getTransactionResult** ： (hash: string)⇒ Promise< null | string > 只在具有存档访问权限并启用了必要的调试 API 的节点上才有效
- **waitForTransaction** : (hash: string, confirms?: number, timeout?: number)⇒ Promise< null | TransactionReceipt > 等待交易完成，可以传入 预设的确认数 以及超时时间（单位 ms）

```javascript
const stx = await storageContract.setScore(95);
const stc = await provider.waitForTransaction(stx.hash);
```

#### 区块相关

- **getBlock**: (blockHashOrBlockTag: BlockTag | string, prefetchTxs?: boolean)⇒ Promise< null | Block > 第一个参数是区块的哈希/区块高度，第二个参数是是否返回当前区块的交易数据， 被放在区块信息的 prefetchedTransactions 属性里
- **getBlockNumber** ： ()⇒ Promise< number > 获取当前区块的高度
- **getTransactionCount** ： (address: AddressLike, blockTag?: BlockTag)⇒ Promise< number > 返回一个地址在连上的交易次数，如果是合约地址，只少有 1 次，因为又一次合约部署交易。如果是用户地址就更多了。如果传入 blockTag, 会计算从链的 0 高度开始到传入这个区块高度内用户/合约的全部交易次数
- **waitForBlock** : (blockTag?: BlockTag)⇒ Promise< Block > 当区块被验证出的时候返回这个区块信息，blockTag 可以是自己预设的一个区块高度， 不如传入 N， 就是要等待 高度为 currentBlockNumber + N 这个区块被挖掘后的区块信息；

#### 日志相关

- **getLogs**: (filter: Filter | FilterByBlockHash)⇒ Promise< Array< Log > > 获取日志， 参数可以是日志对于某个事件的 Filter 也可以是某个区块

```javascript
const logs = await provider.getLogs({
  blockHash,
});
// 或者
const logs = await provider.getLogs(payContract.filters.logReceive());
```

#### 链数据

- **getFeeData** : ()⇒ Promise< FeeData > 返回当前网络的 feeData
- **getNetwork** : ()⇒ Promise< Network > 获取当前链的网络对象
- **lookupAddress** ： (address: string)⇒ Promise< null | string > 根据输入的地址查看 address 对应的 ens 域名，前提是你的链得支持 ENS 注册表，否则报错 Error: network does not support ENS
- **resolveName** ：(ensName: string)⇒ Promise< null | string > 上面方法的逆向方法，输入 ens 域名反查地址

#### 合约相关

- **getStorage** ： (address: AddressLike, position: BigNumberish, blockTag?: BlockTag)⇒ Promise< string > 用来获取指定合约地址在指定[存储槽]()位置， 返回值是一个十六进制的字符串， string 类型的状态不能用 abiCoder.decode, 其他不清楚，uint 和 bool 可以。 如果存储槽没有存任何值，返回的是 0x0.....0 一堆 0
  - address: 合约地址，传入用户地址会发生意外，具体怎么意外不清楚
  - position: 这是一个数字，表示存储槽的位置
  - blockTag: 想要查询的区块，可以是区块高度、区块哈希

```javascript
const storage = await provider.getStorage(storageAddress, 1);
```

## 2 AbstractProvider 类

> 另一个 Provider 实例的描述类，继承自 Provider, ContractRunner, EventEmitterable, NameResolver

### 2.1 构造函数

new AbstractProvider(network?: "any" | Networkish, options?: AbstractProviderOptions)

### 2.2 属性

- **destroyed** : boolean read-only 是否被销毁。 如果销毁了， 所有资源都将被回收，内部事件循环和计时器将被清理，并且不会再向 provider 发送任何请求。
- **disableCcipRead** : boolean 是否关闭 CCIP（跨链操作）的读操作，这里关闭后，用 call 执行合约 pure view 操作打开开启 CCIP read 操作都不好使
- **paused** : boolean 设置为 true 时让处于一种不可用的状态。处于暂停状态时，provider 将不会发送事件，也不会进行任何网络请求
- **plugins** ： Array< AbstractProviderPlugin > 返回注册的插件
- **pollingInterval** : number 轮询间隔
- **provider** ： 返回自己 this

### 2.3 方法

- **\_clearTimeout** : (timerId: number)⇒ void 清理用 this.\_setTimeout 创建的定时器
- **\_detectNetwork** ：()⇒ Promise< Network > 探查当前网络，就是 getNetwork 的功能，子类必须覆写它

## 3 JsonRpcApiProvider 抽象类

> JsonRpcProvider 间接继承自它， 它继承自 Provider 接口 和 AbstractProvider 类

## 4 Eip1193Provider

基于 Eip-1193 的 Provider

- request: (request: { method: string , params?: Array< any > | Record< string, any > })⇒ Promise< any > 允许 DApp 向钱包发出请求，以执行特定的操作，比如获取账户信息、发送交易、签署消息等

## 5 BrowserProvier 类

> 用于封装和提供对浏览器中注入的以太坊提供者（provider）的访问

使用方式

```javascript
const provider = new BrowserProvider(window.ethereum);
```

### 5.1 构造函数参数

- ethereum: 满足 EIP-1193 规范的以太坊对象
- network: 所属网络, 网络 id 或者网络名称
- optins： 配置项
  - cacheTimeout： number 指定缓存的超时时间，单位是 毫秒。这意味着在一定时间内，如果网络或账户信息已经获取，BrowserProvider 会缓存这些信息而不重新请求。如果在超时期限内未获取新的数据，BrowserProvider 会重新请求最新的数据。
  - polling: bool 控制是否启用 轮询 模式。如果启用轮询，BrowserProvider 会定期查询链上的数据（例如账户、区块、交易状态等），以保持数据更新。这对于不依赖事件监听的应用程序很有用，尤其是在用户界面需要频繁更新的场景下。默认 false
  - pollingInterval: number 单位 毫秒， 如果启用了轮询，设置轮询时间
  - staticNetwork： 因为 mateMask 那种钱包会动态切换网络，连接其他链，所以通过设置此选项，可以把网络锁死成自己设置的

### 5.2 方法

- \_send: 用于发送 JSON-RPC 请求的低级接口, 参数是 JsonRPCpayload

```javascript
const payload = {
  jsonrpc: "2.0",
  method: "eth_blockNumber",
  params: [],
  id: 1,
};

const response = await provider._send(payload);
```

- getRpcError: 获取错误

```javascript
try {
  await provider._send(payload);
} catch (error) {
  const rpcError = provider.getRpcError(payload, error);
  console.error(rpcError); // 打印格式化后的错误信息
}
```

- getSigner: 获取 Signer 对象 参数是 Signer 的地址， 如果不填，就是当前 window.ethereum 的钱包地址
- hasSigner: 由于一个 BrowserProvider 可以生成多个 Signer, 所以这里可以判断是否生成了这个 Signer 参数是一个地址，必填
- send: 发送一个标准的 JSON-RPC 请求到以太坊节点。它允许你指定一个 RPC 方法 和对应的 参数，然后返回结果。
  - method: JSON-RPC 方法名， [JSON-RPC api 手册](http://cw.hubwiz.com/card/c/ethereum-json-rpc-api/)
  - param: 给方法传递的参数 看 api 手册里文档传什么参数这里就填什么

## 6 IpcSocketProvider 类

> IPC（Inter-Process Communication）套接字是操作系统提供的一种进程间通信机制。与基于网络的通信方式（如 HTTP 或 WebSocket）不同，IPC 通常用于在同一台机器上的进程之间传输数据。使用 IPC 套接字进行通信的速度通常较快，因为它不需要通过网络传输数据，而是直接在本地进行数据交换。

所以一个 IpcSocketProvider 实例有以下特征：

- **高效的本地连接**：由于 IPC 套接字是本地通信，它通常比通过 HTTP 或 WebSocket 更加高效，延迟较低。
- **仅限于同一台机器**：IPC 连接只能在同一台机器上的节点和脚本之间进行通信，因此客户端（脚本）和节点必须在同一台机器上运行。这意味着你无法像 WebSocket 或 HTTP 那样通过网络远程连接到节点。
- **安全性**：使用 IPC 通信时，数据不会通过网络传输，因此在某些情况下，IPC 可能比其他网络协议（如 HTTP 或 WebSocket）更加安全。

它继承自 SocketProvider, JsonRpcApiProvider 两个类

初始化：
new IpcSocketProvider(path: string, network?: Networkish, options?: JsonRpcApiProviderOptions)

### 6.1 属性

- socket： 连接的 socket

## 2 WebSocketProvider 类

> 继承自 SocketProvider, JsonRpcApiProvider , 通过 websocket 连接生成的 provider

### 2.1 属性

- websocket：WebSocketLike read-only 连接的 websocket 实例
