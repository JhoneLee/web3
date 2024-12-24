# Transport

## 1 Http Transport

> 为 client 创建传输数据函数， 掌管请求

javascript
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const client = createPublicClient({
chain: mainnet,
transport: http('https://eth-mainnet.g.alchemy.com/v2/...'),
})

http 函数的 url 参数如果没有提供，用传输速率差的公共 RPC url

### 1.1 批量请求

Batch JSON-RPC 是一种机制，允许在一次 HTTP 请求中发送多个 JSON-RPC 请求。这意味着你可以将多个请求合并为一个批量请求来提高效率，减少 HTTP 请求的数量。

javascript
const transport = http('https://eth-mainnet.g.alchemy.com/v2/...', {
batch: true
})

### 1.2 参数

> 主要指的就是 http 函数的参数
> [文档](https://viem.sh/docs/clients/transports/http#parameters)

- url : JSON-RPC API 的 url
- batch: bool 或者对象，对象具体配置看文档
- fetchOptions： [参数文档](https://developer.mozilla.org/en-US/docs/Web/API/RequestInit)
- key
- name
- onFetchRequest: (request: Request) => void 当请求发出时的钩子函数，用于记录日志或者 debug
- onFetchResponse: (response: Response) => void 当接收到 response 时候的钩子函数，用于记录日志或者 debug
- retryCount： 重试次数，正整数， 默认是 3
- retryDelay： 单位 ms， 重试的间隔次数
- timeout：单次请求超时时间 默认 10000 单位 ms

## 2 Websocket Transport

> 主要用于连接 wss 连接

javascript
import { createPublicClient, webSocket } from 'viem'
import { mainnet } from 'viem/chains'

const client = createPublicClient({
chain: mainnet,
transport: webSocket('wss://eth-mainnet.g.alchemy.com/v2/...'),
})

如果没有提供就用默认的，影响效率

### 2.1 参数

- url
- keepAlive: bool 或者数值，是否保持连接或保持连接的时间 单位 ms
- key
- name
- reconnect: boolean | { maxAttempts?: number, delay?: number } 连接失败时是否重试 或 重试相关配置
- retryCount： 默认 3
- retryDelay： 默认 150ms
- timeout： 默认 10000ms

## 3 自定义 Transport

使用自定义 Transport 去适配 public client 的时候，你自定义的 provider 一定要具有 public action

使用方式
javascript
import { createWalletClient, custom } from 'viem'
import { mainnet } from 'viem/chains'

const client = createWalletClient({
chain: mainnet,
transport: custom(window.ethereum!)
})

或者

javascript
import { createWalletClient, custom } from 'viem'
import { mainnet } from 'viem/chains'
import { customRpc } from './rpc'

const client = createWalletClient({
chain: mainnet,
transport: custom({
async request({ method, params }) {
const response = await customRpc.request(method, params)
return response
}
})
})

### 3.1 参数

> 主要就是 custom 这个函数的参数
> [指导文档](https://viem.sh/docs/clients/transports/custom#parameters)

- provider： 是一个对象，必须具有一个符合 EIP 1193 规范的 request 函数
- key
- name
- retryCount
- retryDelay

## 4 IPC Transport

IPC（Inter-Process Communication，进程间通信）是一种让不同进程（通常运行在同一台机器上的不同程序）相互通信的机制。简单来说，IPC 允许程序之间交换数据或信号。

在区块链的上下文中，IPC 用于本地进程（比如一个以太坊节点）和客户端（例如你的 dApp 或者脚本）之间的通信。通过 IPC 连接，客户端能够发送 JSON-RPC 请求与节点进行交互，获取区块链数据，发送交易等。

[文档](https://viem.sh/docs/clients/transports/ipc)

javascript
import { createPublicClient } from 'viem'
import { ipc } from 'viem/node'
import { mainnet } from 'viem/chains'

const client = createPublicClient({
chain: mainnet,
transport: ipc('/tmp/reth.ipc'),
})

### 4.1 参数

[文档](https://viem.sh/docs/clients/transports/ipc#parameters)

- path: 可连接 IPC 的系统路径 例如/tmp/reth.ipc
- key
- name
- reconnect
- retryCount
- retryDelay
- timeout

## 5 Fallback Transport

允许客户端在请求失败时自动尝试多个不同的传输方式（例如 HTTP 请求）。如果第一个传输方式失败，客户端会回退到下一个传输方式，直到成功为止。

[文档](https://viem.sh/docs/clients/transports/fallback)

使用方式
javascript
import { createPublicClient, fallback, http } from 'viem'
import { mainnet } from 'viem/chains'

const client = createPublicClient({
chain: mainnet,
transport: fallback([
http('https://eth-mainnet.g.alchemy.com/v2/...'),
http('https://mainnet.infura.io/v3/...')
]),
})

### 5.1 传输排序

通过自动评估每个传输方式的 延迟（latency）和 稳定性（stability），并基于这些评估结果动态地调整 Fallback Transport 中传输方式的优先级。

fallback 的第二个参数是 rank 参数， 可以设置为 true 或者一个对象，设置为 true 的时候，开启默认的排序
javascript
const client = createPublicClient({
chain: mainnet,
transport: fallback([
http('https://eth-mainnet.g.alchemy.com/v2/...'),
http('https://mainnet.infura.io/v3/...')
], { rank: true }),
})

也可以是一个很复杂的对象
javascript
const client = createPublicClient({
chain: mainnet,
transport: fallback(
[
http('https://eth-mainnet.g.alchemy.com/v2/...'),
http('https://mainnet.infura.io/v3/...')
],
{
rank: {
interval: 60_000,
sampleCount: 5,
timeout: 500,
weights: {
latency: 0.3,
stability: 0.7
}
}
}
),
})

### 5.2 参数解释

- rank: [具体文档](https://viem.sh/docs/clients/transports/fallback#rank-optional)
- retryCount
- retryDelay
