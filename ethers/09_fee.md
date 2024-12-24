> 封装了所有和费用有关的操作

### 1.1 构造函数参数

- gasPrice： 单个 gas 的价格
- maxFeePerGas: 每个 gas 的最高限价 EIP-1559 (基础费+优先费)
- maxPriorityFeePerGas：每个 gas 的最高优先价 EIP-1559

### 1.2 属性

- gasPrice：null|bigint readonly 单个 gas 的价格
- maxFeePerGas: null|bigint readonly 每个 gas 的最高限价 EIP-1559 (基础费+优先费)
- maxPriorityFeePerGas：null|bigint readonly 每个 gas 的最高优先价 EIP-1559

### 1.3 方法

- toJSON: 将 feeData 实例转换为 json 数据
