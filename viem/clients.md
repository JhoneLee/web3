# client

> client 相当于 ethers.js 里面的 provider

创建一个 client 需要链信息和 transports 两个元素

- Public Client: 提供对公共操作的访问，例如 getBlockNumber 和 getBalance。这些操作通常是查询区块链状态的信息，不依赖于钱包的状态。
- Wallet Client: 提供对钱包操作的访问，例如 sendTransaction 和 signMessage。这些操作用于发送交易和签署消息，通常需要用户授权或钱包的支持。
- Test Client: 提供对测试操作的访问，例如 mine 和 impersonate (这两个都是矿工验证的工作)。这些操作通常用于本地开发和测试环境，允许模拟区块链行为，比如挖矿或冒充某个账户。

### 1 public client

**基本使用**
javascript
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const publicClient = createPublicClient({
chain: mainnet,
transport: http()
})

**聚合请求**

> 在一个请求内完成多个和链交互的查询操作
> javascript
> const publicClient = createPublicClient({
> batch: {

    multicall: true,  // 启用 multicall 聚合

},
chain: mainnet,
transport: http(),
})

一旦启用了聚合，当你开始使用 readContract 操作时，公共客户端会在消息队列结束时（或者按照你设定的自定义时间段）批量发送这些请求，作为一个单一的 eth_call multicall 请求：
javascript
import { getContract } from 'viem'
import { abi } from './abi'
import { publicClient } from './client'

const contract = getContract({ address, abi, client: publicClient })

// 以下代码将向 RPC 提供者发送一个单一请求。
const [name, totalSupply, symbol, balance] = await Promise.all([
contract.read.name(),
contract.read.totalSupply(),
contract.read.symbol(),
contract.read.balanceOf([address]),
])

**参数解析**
[具体文档](https://viem.sh/docs/clients/public#parameters)

- transport： 传输器， 一般是用 http() 包裹的一个 json rpc url
- chain: 链对象, 从 import { mainnet } from 'viem/chains' 获取
- batch: 批量请求相关设置，是一个对象
  - multicall : 是否开启请求聚合， 是个 bool 还可以是个对象，对聚合进行精细控制，可以看文档
- cacheTime： 缓存数据保留在内存中的时间（以毫秒为单位）
- ccipRead: 跨链读取协议操作是否启用，默认 true， 也有精细操作，具体看文档
- key: 当前 client 的标识符
- name: 当前 client 名称
- pollingInterval： 轮询时间间隔，单位 ms
- rpcSchema ： 设置 RPC schema 的格式 具体看文档

## 2 Wallet Client

> 类似于 ethers 中的 wallet 对象

[文档](https://viem.sh/docs/clients/wallet#wallet-client)

**使用方式**
javascript
import { createWalletClient, custom } from 'viem'
import { mainnet } from 'viem/chains'

const client = createWalletClient({
chain: mainnet,
transport: custom(window.ethereum!)
})

**分为两种**

- 本地钱包: 需要提供私钥或者相关秘钥才能使用的钱包客户端， 秘钥来源有三种
  - 私钥
  - 助记词
  - 层次化确定性（HD）账户
- JSON RPC 钱包: 将交易和消息的签名委托给目标钱包，通过 JSON-RPC 完成, 例如 metaMask 或者满足 EIP-1193 的 window.ethereum 对象

### 2.1 JSON-RPC 钱包

这种类型账户处理简单，只需要使用 window.ethereum 或者 MetaMask 即可

javascript
import { createWalletClient, custom } from 'viem'
import { mainnet } from 'viem/chains'

const client = createWalletClient({
chain: mainnet,
transport: custom(window.ethereum!)
})

const [address] = await client.getAddresses()
const hash = await client.sendTransaction({
account: address,
to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC',
value: parseEther('0.001')
})

如果不希望每个交易都传递 account 字段，可以把 account 提升到 createWalletClient 的位置
javascript
import { createWalletClient, http, parseEther } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

const account = privateKeyToAccount('0x...')

const client = createWalletClient({
account,
chain: mainnet,
transport: http()
})

const hash = await client.sendTransaction({
to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC',
value: parseEther('0.001')
})

### 2.2 本地钱包

声明方式和 JSON-RPC 钱包不同, 主要是 acount 参数来源不同
javascript
import { createWalletClient, http, parseEther } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

const account = privateKeyToAccount('0x...')
const client = createWalletClient({
account,
chain: mainnet,
transport: http()
})

const hash = await client.sendTransaction({
to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC',
value: parseEther('0.001')
})

本地钱包无法使用 public Client 的功能，再创建一个 public client 又很多余，只需要让 local account wallet client 继承 public client 即可

javascript
import { createWalletClient, http, publicActions } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

const account = privateKeyToAccount('0x...')

const client = createWalletClient({
account,
chain: mainnet,
transport: http()
}).extend(publicActions) // 扩展公共操作

// 使用公共操作
const { request } = await client.simulateContract({ ... }) // Public Action

// 使用钱包操作
const hash = await client.writeContract({ ... }) // Wallet Action

### 2.3 参数讲解

[文档](https://viem.sh/docs/clients/wallet#parameters)

- account: 账户信息
- chain：链对象，可以引用 viem/chains 封装好, 也可以[自定义](https://viem.sh/docs/chains/introduction#custom-chains)
- cacheTime：缓存时间 单位 ms
- ccipRead: CCIP 跨链只读协议是否开启， 可以是 bool 也可以是对象，具体看文档
- key: client 的 key
- name: client 的 name
- pollingInterval: 轮询时间间隔，单位 ms， 默认 4s
- rpcSchema： 给 rpc schema 加类型, 类型的定义主要涵盖 json rpc 的方法（例如 eth_getBlockByHash）, 返回值类型、入参类型等。 使用方法 rpcSchema: rpcSchema<CustomRpcSchema>()

## 3 Test Client

使用方式, 相比前两个多了 mode 参数，主要就是模拟挖矿、验证区块的功能
[指导文档](https://viem.sh/docs/clients/test)
javascript
import { createTestClient, http } from 'viem'
import { foundry } from 'viem/chains'

const client = createTestClient({
chain: foundry,
mode: 'anvil',
transport: http(),
})

const mine = await client.mine({ blocks: 1 })

继承前两个 client
javascript
import { createTestClient, http, publicActions, walletActions } from 'viem'
import { foundry } from 'viem/chains'

const client = createTestClient({
chain: foundry,
mode: 'anvil',
transport: http(),
})
.extend(publicActions)
.extend(walletActions)

const blockNumber = await client.getBlockNumber() // Public Action
const hash = await client.sendTransaction({ ... }) // Wallet Action
const mine = await client.mine({ blocks: 1 }) // Test Action

### 3.1 参数

[指导文档](https://viem.sh/docs/clients/test#parameters)

- mode: 测试 client 的种类，取值范围"anvil" | "hardhat" | "ganache" 表示三种本地区块链产品
- transport：传输对象 http('JSON RPC URL')
- account: 测试私钥客户端 privateKeyToAccount('0x...')
- chain: 链对象，支持 hardhat
- cacheTime: 缓存时间 ms
- name: 名称
- pollingInterval: 轮询时间间隔 单位 ms
- rpcSchema

## 4 自定义 client

使用方式
javascript
import { createClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const client = createClient({
chain: mainnet,
transport: http()
})

创建完基础客户端后，你可以根据自己的需求来扩展它，添加更多的自定义功能。这就是为什么在 createClient 后，extend 方法变得非常重要。

javascript
import {
createClient,
http,
formatTransactionRequest,
type CallParameters
} from 'viem'
import { mainnet } from 'viem/chains'

const debugClient = createClient({
chain: mainnet,
transport: http(),
}).extend(client => ({
// ...
async traceCall(args: CallParameters) {
return client.request({
method: 'debug_traceCall',
params: [formatTransactionRequest(args), 'latest', {}]
})
},
// ...
}))

const response = await debugClient.traceCall({
account: '0xdeadbeef29292929192939494959594933929292',
to: '0xde929f939d939d393f939393f93939f393929023',
gas: 69420n,
data: '0xf00d4b5d00000000000000000000000001291230982139282304923482304912923823920000000000000000000000001293123098123928310239129839291010293810'
})
// { failed: false, gas: 69420, returnValue: '...', structLogs: [] }

### 4.1 树摇优化

自定义 client 无需像 public client 和 wallet client 那样全部引入各种 API ， 这里的 getBlock 和 sendTransaction 可以单独引入

[文档](https://viem.sh/docs/clients/custom)

javascript
import { createClient, http } from 'viem'
import { mainnet } from 'viem/chains'
import { getBlock, sendTransaction } from 'viem/actions'

const client = createClient({
chain: mainnet,
transport: http()
})

const blockNumber = await getBlock(client, { blockTag: 'latest' })
const hash = await sendTransaction(client, { ... })

### 4.2 参数

[文档](https://viem.sh/docs/clients/custom#parameters)

- transport
- account
- chain
- batch: 是否开启批量请求，和 public client 一样
- key
- name
- pollingInterval
- rpcSchema
