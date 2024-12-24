# 工具函数

## 1 数字类型

> 区块链世界不用浮点数进行计算，防止精度丢失，在以太坊中，最小单位是 wei， 最大单位是 eth, 1eth = 10\*\*18wei

```javascript
// 将 eth 转换为 wei
eth = parseEther("1.0");
// 1000000000000000000n

// 将数据换换为指定单位的大整数
feePerGas = parseUnits("4.5", "gwei");
// 4500000000n

// 将 wei 转换为 eth (接代码第一行)
formatEther(eth);
// '1.0'

// 将大整数转换为指定单位数字
formatUnits(feePerGas, "gwei");
// '4.5'
```
