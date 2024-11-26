# 合约对象

[ethers.js 合约 API 文档](https://docs.ethers.org/v6/api/contract/)

---

## 1. 合约对象的创建

创建合约对象需要智能合约的 ABI 和地址。如果是只读操作，需要使用 `provider`；如果需要读写操作，需要使用 `signer`。

```javascript
const abi = [
  "function decimals() view returns (string)",
  "function symbol() view returns (string)",
  "function balanceOf(address addr) view returns (uint)",
];

// 创建一个合约对象
const contract = new Contract("dai.tokens.ethers.eth", abi, provider);
```

---

## 2. 合约基类 `BaseContract`

> 合约类 `Contract` 继承自 `BaseContract`

### 2.1 固定属性

- **fallback**: 合约的 `fallback` 函数。
- **filters**: 合约中定义的所有事件，存储为对象形式。
- **interface**: 合约接口，与 `ContractFactory` 的接口一致。
- **runner**: 合约的执行者，可以是 `provider` 对象或 `signer` 对象。
- **target**: 合约的初始地址，可以是普通地址或 ENS 域名。

---

### 2.2 方法

#### 事件及日志相关方法

- **addListener / on**: 添加合约事件监听器。
- **emit**: 触发合约事件。  
  格式: `emit(event: ContractEventName, args: Array<any>) ⇒ Promise<boolean>`
- **off**: 解绑某个事件监听器。
- **once**: 添加一次性触发的事件监听器。
- **queryFilter**: 查询并过滤指定事件的历史日志，可按区块范围筛选。
  ```javascript
  const logs = await contract.queryFilter("Transfer", 0, "latest");
  console.log(logs);
  ```
- **removeAllListeners**: 移除某事件的所有监听器。
- **removeListener**: 移除某事件的指定监听器。

#### 合约创建相关方法

- **attach**: 根据不同的合约地址，在当前合约基础上生成新的合约对象（`runner` 和 `abi` 不变）。
- **connect**: 根据新的 `runner`，生成新的合约对象（`abi` 和 `target` 不变）。

#### 合约部署相关方法

- **deploymentTransaction**: 返回合约部署时生成的交易对象。
- **waitForDeployment**: 等待合约部署完成。

#### 合约元数据相关方法

- **getAddress**: 获取合约实例的地址。
- **getDeployedCode**: 获取合约实例的字节码。
- **getFunction**: 获取合约函数体（适用于函数名与 JS 关键字冲突的情况）。
- **listenerCount**: 返回某事件的监听器数量。
- **listeners**: 返回某事件的所有监听器。

#### 静态方法

- **buildClass**: 根据传入的 ABI 创建新的合约类。

  ```javascript
  const ERC20 = ethers.BaseContract.buildClass(abi);

  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const token1 = new ERC20("0xTokenAddress1", provider);
  const token2 = new ERC20("0xTokenAddress2", provider);
  ```

- **from**: 通过地址（`target`）、ABI 和运行器（`runner`）创建新的 `BaseContract` 实例。
  ```javascript
  const contract = ethers.BaseContract.from(contractAddress, abi, provider);
  ```

---

## 3. `BaseContractMethod` 接口

提供智能合约函数的抽象层，增强类型安全性，方便调用。

### 3.1 属性

- **fragment**: 表示智能合约函数 ABI 的定义。

  ```typescript
  FunctionFragment {
    name: "transfer",
    inputs: [
      { name: "to", type: "address" },
      { name: "amount", type: "uint256" }
    ],
    outputs: [{ name: "", type: "bool" }],
    stateMutability: "nonpayable"
  }
  ```

- **name**: 合约函数的名称。

### 3.2 方法

- **estimateGas**: 评估函数调用所需的 Gas。
- **getFragment**: 根据参数返回带参数的 ABI 片段。
- **populateTransaction**: 生成交易对象，但不会发送。
- **send**: 执行修改状态的交易。
- **staticCall**: 模拟函数调用，不发送交易，也不消耗 Gas。
- **staticCallResult**: 类似 `staticCall`，但返回结果不同。
  ```javascript
  const result = await contract.getUserName.staticCall(); // 返回值直接提炼
  const detailedResult = await contract.getUserName.staticCallResult(); // 返回详细结果 {0: "jack"}
  ```

---

## 4. `ContractFactory` 类

用于在开发框架（如 Hardhat）中部署合约。

### 4.1 属性

- **bytecode**: 被部署合约的字节码。
- **interface**: 合约的接口对象。
- **runner**: 合约工厂的执行者。

### 4.2 构造函数

参数：

- **abi**: 必填，合约的 ABI。
- **bytecode**: 必填，合约的字节码。
- **runner**: 选填，执行者。

### 4.3 方法

- **attach**: 根据地址生成对应的合约实例。
- **connect**: 使用新的 `runner`。
- **deploy**: 部署合约，参数为构造函数参数。
- **getDeployTransaction**: 获取部署交易对象。

### 4.4 静态方法

- **fromSolidity**: 从 ABI 和字节码创建 `ContractFactory` 实例。
  ```javascript
  const factory = ethers.ContractFactory.fromSolidity({ abi, bytecode });
  const contract = await factory.connect(wallet).deploy();
  ```

---

## 5. `ContractTransaction` 接口

### 属性

- **data**: 交易数据。
- **from**: 交易发起者地址。
- **to**: 交易接收者地址。

---

## 6. `EventLog` 类

### 属性

- **args**: 事件参数，已解析。
- **eventName**: 事件名称。
- **eventSignature**: 事件签名。
- **fragment**: 事件的 Fragment 对象。
- **interface**: 事件的 Interface 对象。
