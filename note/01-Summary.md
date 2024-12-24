# 汇总

# 3 Contract

[详细文档](https://github.com/JhoneLee/web3/blob/main/note/03-Contract.md)

## 3.1 类和实例

- class BaseContract 合约基类, 用于实例化合约对象 需要合约地址、abi 和 provide 或 signer
- 合约实例：
  - 可以获取合约相关的（本身、事件、参数）的`abi`信息
  - 监听/解绑事件，过滤日志
  - 获取合约元数据
  - 更改当前合约 地址 或者 runner
- class ContractFactory 合约工厂，用于通过 js 脚本部署智能合约 需要合约的 abi bytecode
- 合约工厂实例： 部署合约，取得部署合约的交易对象
- class EventLog: 合约事件日志类

## 3.2 接口

- ContractTransaction：定义了合约交易数据的结构
- EventLog: 合约事件日志的数据结构
- BaseContractMethod：补充了一些合约实例的方法
