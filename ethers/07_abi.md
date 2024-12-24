### 1. AbiCoder

`AbiCoder` 是处理 ABI 编码与解码的底层类，提供了用于将 JavaScript 变量转换为 ABI 编码格式，或将 ABI 编码数据转换为 JavaScript 变量的方法。

#### 方法：

- **decode**：将 ABI 编码的二进制数据解码为 JavaScript 变量。支持松散解码（loose）以兼容老版本 Solidity 编译器生成的数据。
  ```javascript
  abiCoder.decode(["string"], result);
  ```
- **encode**：根据给定类型编码数据为 ABI 格式，返回十六进制字符串。

  ```javascript
  const encodedData = abiCoder.encode(["uint256"], [0n]);
  ```

- **getDefaultValue**：根据类型返回默认值，例如对于一个 `uint256` 类型会返回 `0`。

#### 静态函数：

- **\_setDefaultMaxInflation**：设置 ABI 编码器的最大膨胀值，影响解码时的填充长度（不常用）。
- **getBuiltinCallException**：用于解析智能合约调用失败时的异常数据。

### 2. Fragment 抽象类

`Fragment` 是 ABI 中的最小单位，表示智能合约的不同元素（如函数、事件、构造函数等）。

#### 属性：

- **inputs**：表示 ABI 中的输入参数类型。
- **type**：指定元素的类型，如 `"constructor"`、`"event"`、`"function"` 等。

#### 方法：

- **format**：将 Fragment 对象格式化为不同格式，支持 `sighash`（函数签名哈希）、`minimal`（简化 ABI）、`full`（完整 ABI）和 `json`（JSON 格式）。

#### 静态方法：

- **isContructor**, **isError**, **isEvent**, **isFunction**, **isStruct**：用于判断传入的 Fragment 是否属于某一类型（例如函数、构造函数等）。

### 3. FunctionFragment

`FunctionFragment` 继承自 `Fragment`，专门用于描述合约函数的 ABI。

#### 属性：

- **constant**：判断函数是否为 `pure` 或 `view` 类型（即不修改链上状态的函数）。
- **gas**：推荐的 gas 限制。
- **outputs**：函数的返回值类型。
- **payable**：判断函数是否可以接收 ETH。
- **selector**：函数选择器，用于识别调用的函数。
- **stateMutability**：函数的状态可变性，取值包括 `"payable"`, `"nonpayable"`, `"view"`, `"pure"`。

#### 函数：

- **format**：将 `FunctionFragment` 格式化为不同的输出格式。

#### 静态方法：

- **getSelector**：通过函数名和参数获取函数的选择器。

  ```javascript
  const funcSelector = storageContract.getFunction("setName").fragment.selector;
  ```

- **isFragment**：判断传入的对象是否是一个 `FunctionFragment`。

### 4. EventFragment

`EventFragment` 用于描述事件的 ABI。

#### 属性：

- **anonymous**：事件是否为匿名事件。
- **topicHash**：事件签名的 `topic` 哈希值。

#### 方法：

- **getTopicHash**：返回事件的 `topic` 哈希，通常用于日志过滤。

### 5. ErrorFragment, ConstructorFragment, FallbackFragment, StructFragment

这些类与 `FunctionFragment` 类似，但分别用于描述合约中的自定义错误（`ErrorFragment`）、构造函数（`ConstructorFragment`）、支付回调（`FallbackFragment`）和结构体（`StructFragment`）。它们缺少某些特定于函数的属性和方法，但具有与 `FunctionFragment` 相似的功能。
