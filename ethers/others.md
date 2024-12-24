## **1. 智能合约的调用**

智能合约调用通常分为读取（只读）和写入（状态改变）两种操作，分别使用 `provider` 和 `signer` 来创建合约实例。

### **1.1 创建合约对象**

合约对象是通过智能合约的 **ABI**（应用二进制接口）和合约地址来创建的。如果是只读操作（不改变区块链状态），需要使用 `provider`。如果需要修改状态，发送交易，则需要使用 `signer`。

```javascript
// 合约的 ABI 和地址
const abi = [
  "function decimals() view returns (string)",
  "function symbol() view returns (string)",
  "function balanceOf(address addr) view returns (uint)",
];

// 使用 provider 创建合约实例（只读操作）
const contract = new ethers.Contract("dai.tokens.ethers.eth", abi, provider);
```

### **1.2 调用合约函数**

**示例：调用 `setScore` 方法来修改状态**

假设合约有一个状态变量 `score` 和一个方法 `setScore(uint inputScore)`，你可以通过合约对象来调用这个方法。

#### **传统调用方式：**

```javascript
// 合约的 ABI 和地址
const contractAbi = ["function setScore(uint inputScore) public"];
const contractAddress = "0xYourContractAddress";

// 创建合约实例
const contract = new ethers.Contract(contractAddress, contractAbi, signer);

// 调用 setScore 方法，传入 100
const tx = await contract.setScore(100);
```

在这个例子中，`ethers.Contract` 会自动创建符合 `TransactionRequest` 接口的数据，并发送交易。

#### **自定义交易请求对象：**

如果你想手动构建交易请求对象，可以使用合约的 `Interface` 来编码函数数据。

```javascript
// 获取合约的 Interface 对象
const contractInterface = new ethers.Interface(contractAbi);

// 编码函数调用数据
const data = contractInterface.encodeFunctionData("setScore", [100]);

// 构建 TransactionRequest 对象
const transactionRequest = {
  to: contractAddress, // 合约地址
  data: data, // 编码后的 ABI 数据
  gasLimit: 21000, // 设置 gas 限制（可以使用 estimateGas 方法来自动估算）
  nonce: await signer.getTransactionCount(), // 获取当前的 nonce，防止重放攻击
  value: 0, // 合约不需要转账 ETH，所以是 0
  chainId: await provider.getNetwork().then((network) => network.chainId), // 获取当前网络的链 ID
};

// 手动发送交易
const txResponse = await signer.sendTransaction(transactionRequest);
```

#### **重要概念：**

- **`Interface.encodeFunctionData`**: 这个方法用于编码函数调用的参数。它将函数名和参数按照 ABI 标准转换成一个字符串数据，可以直接用于交易请求的 `data` 字段。
- **`TransactionRequest`**: 一个对象，描述了交易的细节，包括目标地址、数据、gas、nonce 等。你可以自定义这些字段，进行精细的控制。
