# bchaintrade接口文档
## 前台接口
### 1. 仓库管理员相关接口
#### 1.1 申请开通仓库管理员账户第一步
```http
POST /bchaintrade/storekeepers/apply_first
```
  
> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| email | string | 是 | 邮箱 |  |
| fullname | string | 是 | 姓名 |  |
  
> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": null
}
```
> ##### 说明
> 执行成功后`status` = 1
> 失败原因：邮箱不合法、其他用户已使用此邮箱、邮箱不能为空、姓名不能为空

#### 1.2 申请开通仓库管理员账户第二步
```http
POST /bchaintrade/storekeepers/apply_second
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| xx | string | 是 | |  |
| xx | string | 是 | |  |

#### 1.3 获取仓库管理员账户申请状态(直接获取info?)
```http
GET /bchaintrade/storekeeper/status
```
  
  > ##### 请求参数
  空
  
  > ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "status": 0
  }
}
```
  > <a name="storekeeper_status"></a>
  >
| status | 说明 |
| ------ | --- |
| 0 | 未开通 |
| 1 | 申请中，已添加仓储账号 |
| 2 | 申请中，已上传材料 |
| 3 | 审核中 |
| 4 | 已开通 |


#### 1.4 获取仓库管理员账户信息
```http
GET /bchaintrade/storekeeper/info
```

> ##### 请求参数
空

> ##### 返回结果
```josn
{
  "code": 0,
  "msg": "成功",
  "data": {
    "status": 4,
    "email": "storekeeper@smm.cn",
    "email_status": 0,
    "fullname": "张三",
    "bchain_account": [
      "0xbc3eac5435d146e7bdca20144d52bab6ab503af1",
      "0xbc3eac5435d146e7bdca20144d52bab6ab503af2"
    ]
  }
}
```
> `status`: 上文已给出[说明](#storekeeper_status)
> 
| email_status | 说明 |
| ------ | --- |
| 0 | 未激活 |
| 1 | 已激活 |
> 

#### 1.5 修改仓库管理员账户信息
```http
POST /bchaintrade/storekeeper/update
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| email | string | 是 | 邮箱 | 更改邮箱 |

> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "status": 4,
    "email": "storekeeper2@smm.cn",
    "email_status": 0,
    "fullname": "张三",
    "bchain_account": [
      "0xbc3eac5435d146e7bdca20144d52bab6ab503af1",
      "0xbc3eac5435d146e7bdca20144d52bab6ab503af2"
    ]
  }
}
```


#### 1.6 仓库管理员账户邮箱验证
```http
GET /bchaintrade/storekeeper/email_verify
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| token | string | 是 |  | jwt(email+时间戳) |

> ##### 返回结果
页面跳转至会员中心首页

> ##### 实现
redis:  email_verify:email:时间戳 user_id

#### 1.7 网上交易申请列表
```http
GET /bchaintrade/storekeeper/trading_application_list
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| page_index | int | 否 | 当前页数 | 默认值：1 |
| page_items | int | 否 | 单页显示项数 | 默认值：10 |
| field | string | 否 | ["company_name", "goods_name", "warehouse_receipt_num"] | 企业名称、货物名称、存货凭证号 |
| value | string | 否 | 开始时间 |  |
| time_end | string | 否 | 结束时间 |  |

> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "page_index": 1,
    "page_items": 10,
    "amount": 100,
    "items": [
      {
        "id": 1,
        "serial_num": "B001001001",
        "create_time": "1470109581",
        "company_name": "上海某某企业公司",
        "goods_name": "电解铜",
        "warehouse_receipt_num": "BH-123456",
        "quantity": 5,
        "quantity_unit": "捆",
        "net_weight": 5.0000,
        "weight_unit": "吨",
        "status": 1
      },
      {
        "id": 2,
        "serial_num": "B001001001",
        "create_time": "1470109581",
        "company_name": "上海某某企业公司",
        "goods_name": "电解铜",
        "warehouse_receipt_num": "BH-123456",
        "quantity": 5,
        "quantity_unit": "捆",
        "net_weight": 5.0000,
        "weight_unit": "吨",
        "status": 1
      },
      ...
    ]
  }
}
```

> 
| status | 说明 |
| ------ | --- |
| 0 | 待审核（卖家未确认，仓库未确认） |
| 1 | 待审核（卖家已确认，仓库未确认） |
| 2 | 已拒绝 |
| 3 | 已通过 |

#### 1.8 网上交易申请详情
```http
GET /bchaintrade/storekeeper/trading_application
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| id | int | 否 |  |  |

> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "id": 1,
    "serial_num": "B001001001",
    "create_time": "1470109581",
    "company_name": "上海某某企业公司",
    "goods_name": "电解铜",
    "warehouse_receipt_num": "BH-123456",
    "warehouse_receipt_img": "http://test.jpg",
    "quantity": 5,
    "quantity_unit": "捆",
    "net_weight": 5.0000,
    "gross_weight": 6.0000,
    "weight_unit": "吨",
    "status": 1,
    "warehouse_num": "C001123-123456",
    "warehouse_name": "上海某某仓储服务有限公司",
    "warehouse_addr": "上海市浦东新区峨山路91弄20号陆家嘴软件园9号楼北塔8层",
    "company_user": "张三",
    "company_confirm_time": "1470109581"
  }
}
```

#### 1.9 网上交易撤销列表
```http
GET /bchaintrade/storekeeper/trading_cancellation_list
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| page_index | int | 否 | 当前页数 | 默认值：1 |
| page_items | int | 否 | 单页显示项数 | 默认值：10 |
| field | string | 否 | ["company_name", "goods_name", "warehouse_receipt_num"] | 企业名称、货物名称、存货凭证号 |
| value | string | 否 | 开始时间 |  |
| time_end | string | 否 | 结束时间 |  |


> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "page_index": 1,
    "page_items": 10,
    "amount": 100,
    "items": [
      {
        "id": 1,
        "serial_num": "B001001001",
        "create_time": "1470109581",
        "company_name": "上海某某企业公司",
        "goods_name": "电解铜",
        "warehouse_receipt_num": "BH-123456",
        "quantity": 5,
        "quantity_unit": "捆",
        "net_weight": 5.0000,
        "weight_unit": "吨",
        "status": 1
      },
      {
        "id": 2,
        "serial_num": "B001001001",
        "create_time": "1470109581",
        "company_name": "上海某某企业公司",
        "goods_name": "电解铜",
        "warehouse_receipt_num": "BH-123456",
        "quantity": 5,
        "quantity_unit": "捆",
        "net_weight": 5.0000,
        "weight_unit": "吨",
        "status": 1
      },
      ...
    ]
  }
}
```

> 
| status | 说明 |
| ------ | --- |
| 0 | 待审核（卖家未确认，仓库未确认） |
| 1 | 待审核（卖家已确认，仓库未确认） |
| 2 | 已拒绝 |
| 3 | 已通过 |

#### 1.8 网上交易申请详情
```http
GET /bchaintrade/storekeeper/trading_cancellation
```

> ##### 请求参数
| 参数 | 类型 | 必填 | 说明 | 备注 |
| ---- |---- | ---- | ---- | ---- |
| id | int | 否 |  |  |

> ##### 返回结果
```json
{
  "code": 0,
  "msg": "成功",
  "data": {
    "id": 1,
    "serial_num": "B001001001",
    "create_time": "1470109581",
    "company_name": "上海某某企业公司",
    "goods_name": "电解铜",
    "warehouse_receipt_num": "BH-123456",
    "warehouse_receipt_img": "http://test.jpg",
    "quantity": 5,
    "quantity_unit": "捆",
    "net_weight": 5.0000,
    "gross_weight": 6.0000,
    "weight_unit": "吨",
    "status": 1,
    "warehouse_num": "C001123-123456",
    "warehouse_name": "上海某某仓储服务有限公司",
    "warehouse_addr": "上海市浦东新区峨山路91弄20号陆家嘴软件园9号楼北塔8层",
    "company_user": "张三",
    "company_confirm_time": "1470109581"
  }
}
```

### 2. 仓库管理

## 后台接口
### 1. 账户管理
### 2. 仓库管理
