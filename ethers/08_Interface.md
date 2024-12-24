# Interface

## 1. Interface 类

`Interface` 类用于将 ABI 数据结构转化为 JavaScript 中的格式，方便开发者与智能合约交互。它帮助开发者进行合约函数的调用、事件监听、返回值解析以及错误处理。

### 1.1 创建实例

`Interface` 可以通过两种方式进行声明：

```javascript
const payInterface = new ethers.Interface(payAbi);
const storageInterface = ethers.Interface.from(storageAbi);
```

其中 `payAbi` 和 `storageAbi` 是智能合约的 ABI 描述。

### 1.2 属性

`Interface` 类包含一些常用的属性：

- **deploy**: `ConstructorFragment`，只读，返回智能合约的构造函数 ABI 描述片段。
- **fallback**: `FallbackFragment | null`，只读，返回合约的 fallback 函数的 ABI 描述片段。
- **fragments**: `ReadonlyArray<Fragment>`，只读，列出智能合约的所有片段，包括函数、事件等。
- **receive**: `boolean`，只读，判断智能合约是否声明了 `receive` 函数。

### 1.3 方法

`Interface` 类提供了很多方法来帮助开发者对智能合约的 ABI 进行编码、解码、解析等操作。

#### 1.3.1 解码方法

- **decodeErrorResult**:
  解码合约调用失败时的错误结果数据。

  ```javascript
  interface.decodeErrorResult(fragment, data);
  ```

- **decodeEventLog**:
  解码事件日志数据。可以指定 `topics` 来过滤事件。

  ```javascript
  interface.decodeEventLog(fragment, data, topics);
  ```

- **decodeFunctionData**:
  解码函数调用数据。`data` 通常来源于交易的 `data` 字段。
  ```javascript
  interface.decodeFunctionData(fragment, data);
  ```

#### 1.3.2 编码方法

- **encodeDeploy**:
  编码合约部署的数据，通常用于交易的 `data` 字段。

  ```javascript
  interface.encodeDeploy(values);
  ```

- **encodeErrorResult**:
  编码合约调用失败的错误结果。

  ```javascript
  interface.encodeErrorResult(fragment, values);
  ```

- **encodeEventLog**:
  编码事件日志数据，包括 `data` 和 `topics`。

  ```javascript
  const { data, topics } = interface.encodeEventLog(fragment, values);
  ```

- **encodeFunctionData**:
  编码函数调用数据（交易的 `data` 字段）。

  ```javascript
  const data = interface.encodeFunctionData(fragment, values);
  ```

- **encodeFunctionResult**:
  编码函数返回值。
  ```javascript
  const encodedResult = interface.encodeFunctionResult(fragment, values);
  ```

#### 1.3.3 遍历方法

- **forEachError**:
  遍历所有的错误片段，并执行回调函数。

  ```javascript
  interface.forEachError((func, index) => {
    console.log(func, index);
  });
  ```

- **forEachEvent**:
  遍历所有的事件片段，并执行回调函数。

  ```javascript
  interface.forEachEvent((func, index) => {
    console.log(func, index);
  });
  ```

- **forEachFunction**:
  遍历所有的函数片段，并执行回调函数。
  ```javascript
  interface.forEachFunction((func, index) => {
    console.log(func, index);
  });
  ```

#### 1.3.4 格式化方法

- **format**:
  将 ABI 编码格式化为可读格式。如果传入 `true`，返回最精简的格式。

  ```javascript
  const formattedAbi = interface.format();
  ```

- **formatJson**:
  将 ABI 格式化为 JSON 字符串。
  ```javascript
  const abiJson = interface.formatJson();
  ```

#### 1.3.5 错误和事件解析

- **getError**:
  根据错误签名或名称获取错误片段。

  ```javascript
  const errorFragment = interface.getError(key, values);
  ```

- **getEvent**:
  根据事件签名或名称获取事件片段。

  ```javascript
  const eventFragment = interface.getEvent(key, values);
  ```

- **getEventName**:
  根据事件的 `topic` 哈希或签名获取事件名称。

  ```javascript
  const eventName = interface.getEventName(key);
  ```

- **getFunction**:
  根据函数签名或名称获取函数片段。

  ```javascript
  const functionFragment = interface.getFunction(key, values);
  ```

- **getFunctionName**:
  根据函数签名或选择器获取函数名称。
  ```javascript
  const functionName = interface.getFunctionName(key);
  ```

#### 1.3.6 存在性检查

- **hasEvent**:
  判断 ABI 中是否存在指定的事件。

  ```javascript
  const exists = interface.hasEvent(key);
  ```

- **hasFunction**:
  判断 ABI 中是否存在指定的函数。
  ```javascript
  const exists = interface.hasFunction(key);
  ```

#### 1.3.7 解析方法

- **parseCallResult**:
  解析函数调用的执行结果（返回值）。

  ```javascript
  const result = interface.parseCallResult(data);
  ```

- **parseError**:
  解析错误数据。

  ```javascript
  const error = interface.parseError(data);
  ```

- **parseLog**:
  解析日志数据（`data` 和 `topics`）。

  ```javascript
  const log = interface.parseLog(log);
  ```

- **parseTransaction**:
  解析交易数据（`data` 和 `value`）。
  ```javascript
  const transaction = interface.parseTransaction(tx);
  ```
