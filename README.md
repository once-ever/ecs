# EOS Smart Contract Standard
此项目旨在定提供一个EOS Token类合约开发的参考流程，包含合约设计、审计以及上线等。

## 合约设计

### 原则
1. Token类合约需要开源。如需使用第三方库，则必须使用开源库，合约编译时直接使用源代码进行编译。
2. Token类合约需要相对独立，不能包含业务逻辑。如果业务需要有token操作，应通过合约接口进行交互。

### 接口设计
Token类合约必须包含以下接口
```
void create( account_name issuer, asset maximum_supply );
void issue( account_name to, asset quantity, string memo );
void transfer( account_name from, account_name to, asset quantity, string memo );
```
**说明**
1. 对于transfer接口，要求所有资产转移必须在链上可查询，包括用于余额变化、收取交易费用等。
2. Token的symbol需满足EOS对symbol的要求
    1 必须全部是大写字母（A-Z）
    2 长度小于等于7位
3. 对于有发行进度限制的token，建议在issue接口中进行限制
4. 对于有锁定功能的token，建议在accounts表（见数据表设计）中同时记录用户余额和可用余额，并提供接口进行查询

### 数据表设计
Token类合约必须包含accounts表和stat表，分别提供用户余额查询功能和发行状态查询功能

对于accounts表，典型的表结构定义如下
```
struct account {
    asset    balance;
    
    uint64_t primary_key()const { return balance.symbol.name(); }
};
```
1. 表名称必须为accounts
2. 至少包含用户余额字段
3. primary key必须为token的symbol name
4. 调用表的构造函数时，code必须为合约账户(_self), scope必须为用户账户

对于stat表，典型的表结构定义如下
```
struct currency_stats {
    asset          supply;
    asset          max_supply;
    account_name   issuer;
    uint64_t primary_key()const { return supply.symbol.name(); }
};
```
1. 表名称必须为stat
2. 至少包含以下字段
    1 发行总量
    2 已发行量
    3 发行人
3. primary key必须为token的symbol name
4. 调用表的构造函数时，code必须为合约账户(_self), scope必须为symbol name

**说明**
1. 表名称的长度必须小于等于12位
2. 定义表结构的结构体名称长度建议小于等于12位（下划线除外）。

## 审计
