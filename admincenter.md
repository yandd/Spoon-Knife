admincenter设计文档
==============

<!-- MarkdownTOC depth=3 -->

- [说明](#说明)
- [MySQL表设计](#MySQL表设计)
  - [用户表 `auth_user`](#用户表-auth_user)
  - [角色表 `auth_role`](#角色表-auth_role)
  - [用户角色表 `auth_user_roles`](#用户角色表-auth_user_roles)
  - [应用表 `auth_app`](#应用表-auth_app)
  - [应用模块表 `auth_app_modules`](#应用模块表-auth_app_modules)
  - [角色权限表 `auth_role_permissions`](#角色权限表-auth_role_permissions)
- [代码设计](#代码设计)
  - [代码示例](#代码示例)
  - [jwt设计](#jwt设计)
  - [API设计](#API设计)
- [附录](#附录)
  - [错误代码说明](#错误代码说明)

<!-- /MarkdownTOC -->

## 说明
* `user`分为`super_user`和`normal_user`
* `user`可以被禁用
* `hyk_center`,`mall_center`...统称为`app`, 每个`app`有不同的`module`
* `module`对应的操作方法有`CURD(Create, Update, Read, Delete)`(可以扩展,例如`AddChild`, `DelChild`),当所表述`module`超出这四种方法,需重新划分`module`直到能用这四种方法来操作为止(通用权限设计?);
* `CURD`对应有四种操作权限:
  - `b'0001'` -> `Create`
  - `b'0010'` -> `Update`
  - `b'0100'` -> `Read`
  - `b'1000'` -> `Delete`

  权限可以通过或运算组合,例如`permission`取值为`b'0101'`表示`(Create | Read)`,即同时拥有创建和读取`module`的权限
* `user`添加`role`后,则继承了`role`对`module`的操作权限,一个`user`可以添加多个`role`

## MySQL表设计

### 用户表 `auth_user`

```sql
CREATE TABLE `auth_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `email` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `password` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `fullname` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `department` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `is_super` tinyint(1) NOT NULL DEFAULT '0',
  `is_active` tinyint(1) NOT NULL DEFAULT '1',
  `is_del` tinyint(1) NOT NULL DEFAULT '0',
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_login_ip` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `last_login_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_user_email_uniq` (`email`),
  KEY `auth_user_fullname` (`fullname`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

* `is_super` : `0`, 表示非超级用户; `1`,表示超级用户
* `is_active` : `0`,表示禁用;`1`,表示启用

### 角色表 `auth_role`
```sql
CREATE TABLE `auth_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `summary` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_role_name_uniq` (`name`),
  KEY `auth_role_title` (`title`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 用户角色表 `auth_user_roles`
```sql
CREATE TABLE `auth_user_roles` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `role_id` int(11) NOT NULL,
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_user_roles_user_id_role_id_uniq` (`user_id`,`role_id`),
  KEY `auth_user_roles_role_id` (`role_id`),
  CONSTRAINT `auth_user_roles_role_id_fk_auth_role_id` FOREIGN KEY (`role_id`) REFERENCES `auth_role` (`id`) ON DELETE CASCADE,
  CONSTRAINT `auth_user_roles_user_id_fk_auth_user_id` FOREIGN KEY (`user_id`) REFERENCES `auth_user` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 应用表 `auth_app`
```sql
CREATE TABLE `auth_app` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `title` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `order` int(11) NOT NULL DEFAULT '0',
  `is_active` tinyint(1) NOT NULL DEFAULT '1',
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_app_name_uniq` (`name`),
  KEY `auth_app_title` (`title`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 应用模块表 `auth_app_modules`
```sql
CREATE TABLE `auth_app_modules` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `app_id` int(11) NOT NULL,
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `title` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `order` int(11) NOT NULL DEFAULT '0',
  `is_active` tinyint(1) NOT NULL DEFAULT '1',
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_app_modules_app_id_name_uniq` (`app_id`,`name`),
  KEY `auth_app_modules_app_id` (`app_id`),
  KEY `auth_app_modules_title` (`title`),
  CONSTRAINT `auth_app_modules_app_id_fk_auth_app_id` FOREIGN KEY (`app_id`) REFERENCES `auth_app` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 角色权限表 `auth_role_permissions`
```sql
CREATE TABLE `auth_role_permissions` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `role_id` int(11) NOT NULL,
  `module_id` int(11) NOT NULL,
  `permission` bigint(11) NOT NULL DEFAULT '0',
  `created_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_by` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `updated_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auth_role_permissions_role_id_module_id_uniq` (`role_id`,`module_id`),
  KEY `auth_role_permissions_module_id_uniq` (`module_id`),
  CONSTRAINT `auth_role_permissions_module_id_fk_auth_app_module_id` FOREIGN KEY (`module_id`) REFERENCES `auth_app_modules` (`id`) ON DELETE CASCADE,
  CONSTRAINT `auth_role_permissions_role_id_fk_auth_role_id` FOREIGN KEY (`role_id`) REFERENCES `auth_role` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## 代码设计

### 代码示例

```go
if user.is_super() || user.has_perm("app_name", "module_name", READ_PERM | DELETE_PERM) {
}
```

### jwt设计
```json
{
    "id": 1,
    "email": "test@smm.cn",
    "fullname": "测试帐号",
    "create_at": unix_timestamp,
    "expire_at": unix_timestamp,
    "is_super": true,
    "perm": {
        "app1": {
          "module1": 7,       // b'0111' -> (Create | Update | Read)
          "module2": 4        // b'0100' -> Read
        }
    }
}
```

### API设计

* #### 用户登陆
  ```
  POST /admin/user/login
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | username |string|是|邮箱||
  | password |string|是|密码||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "msg": "成功",
    "data": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  }
  ```

* #### 刷新token
  ```
  POST /admin/token/refresh
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "msg": "成功",
    "data": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  }
  ```

* #### 修改密码
  ```
  POST /admin/user/modify_password
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | old_password |string|是|原密码||
  | new_password |string|是|新密码||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "msg": "成功",
    "data": null
  }
  ```

* #### 重置用户密码(以邮件方式发送新密码)
  ```
  POST /admin/user/reset_password
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id |int|是|用户ID||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "msg": "成功",
    "data": null
  }
  ```

* #### 获取用户列表
  ```
  GET /admin/user/list
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | page_index | int | 否 | 分页页码 | 默认为 1 |
  | page_items | int | 否 | 分页项数 | 默认为 10 |
  | field | enum{"email", "fullname"} | 否 | 查询字段 | |
  | value | string | 否 | 查询值 | |
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": [
      {
        "id": 9,
        "email": "dongchen@smm.cn",
        "fullname": "dongchen",
        "roles": "控制中心管理员、支付系统管理员、账户中心管理员",
        "is_active": 1,
        "last_login_time": "2016-06-22 15:53:35",
        "last_login_ip": ""
      },
      ...
    ],
    "msg": "成功"
  }
  ```

* #### 获取用户数
  ```
  GET /admin/user/list/count
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | field | enum{"email", "fullname"} | 否 | 查询字段 | |
  | value | string | 否 | 查询值 | |
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "count": 9
    },
    "msg": "成功"
  }
  ```

* #### 获取用户信息
  ```
  POST /admin/user/info
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 用户ID ||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 10,
      "email": "yandongdong@smm.cn",
      "fullname": "严东东",
      "roles": "控制中心管理员、支付系统管理员",
      "is_active": 1,
      "last_login_time": "2016-06-22 17:51:53",
      "last_login_ip": ""
    },
    "msg": "成功"
  }
  ```

* #### 添加用户
  ```
  POST /admin/user/add
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | email |string|是|邮箱||
  | fullname | string |否|姓名||
  | department | string |否|部门||
  | role[] | array | 否 |角色| [role_id1, role_id2, ...] |

  > #### 请求示例
  ```
  /admin/user/add?email=yandongdong@smm.cn&fullname=严东东&department=IT&role[]=1&role[]=2
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 10,
      "email": "yandongdong@smm.cn",
      "fullname": "严东东",
      "roles": "控制中心管理员、支付系统管理员",
      "is_active": 1,
      "last_login_time": "2016-06-22 17:51:53",
      "last_login_ip": ""
    },
    "msg": "成功"
  }
  ```

* #### 更新用户(开启/关闭用户、设定用户角色)
  ```
  POST /admin/user/update
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 用户ID | |
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |
  | role[] | array | 否 |角色| [role_id1, role_id2, ...] |

  > ##### 说明
  ```
  jquery $.get 传入参数:
  {
    "role": [
      1, 3
    ]
  }
  对应的URL: role[]=1&role[]=3
  ```

  > ##### 请求示例
  ```
  /admin/user/update?id=10&is_active=0&role[]=1&role[]=3
  /admin/user/update?id=10&is_active=0
  /admin/user/update?id=10&role[]=1&role[]=3
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 10,
      "email": "yandongdong@smm.cn",
      "fullname": "严东东",
      "roles": "控制中心管理员、支付系统管理员",
      "is_active": 0,
      "last_login_time": "2016-06-22 17:51:53",
      "last_login_ip": ""
    },
    "msg": "成功"
  }
  ```

* #### 删除用户
  ```
  POST /admin/user/del
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 用户ID ||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": null,
    "msg": "成功"
  }
  ```

* #### 获取角色列表
  ```
  GET /admin/role/list
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | field | enum{"name", "summary"} | 否 | 查询字段,summary为模糊搜索 | |
  | value | string | 否 | 查询值 | |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": [
      {
        "id": 1,
        "name": "控制中心管理员",
        "summary": "",
        "user_count": "10",
        "created_by": "",
        "created_time": "2016-06-22 16:10:20",
        "perm": null
      },
      ...
    ],
    "msg": "成功"
  }
  ```

* #### 获取角色信息
  ```
  GET /admin/role/info
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 角色ID | |
  | with_perm | enum{0, 1} | 否 | 是否返回权限 | 0: 不返回权限, 1: 返回权限 |

  > ##### 说明
  ```
  返回的权限说明 请参考下文中的 更新角色
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 4,
      "name": "商城系统管理员",
      "summary": "",
      "user_count": "0",
      "created_by": "test@smm.cn",
      "created_time": "2016-06-22 18:26:15",
      "perm": {
        "1": 4,
        "2": 7,
        "3": 15
      }
    },
    "msg": "成功"
  }
  ```

* #### 添加角色
  ```
  POST /admin/role/add
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | name |string|是|名称||
  | summary | string |否|摘要||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 4,
      "name": "商城后台管理员",
      "summary": "",
      "user_count": "0",
      "created_by": "test@smm.cn",
      "created_time": "2016-06-22 18:26:15",
      "perm": null
    },
    "msg": "成功"
  }
  ```

* #### 更新角色(修改角色名称、设定角色权限)
  ```
  POST /admin/role/update
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 角色ID ||
  | name | string | 否 | 角色名称 ||
  | perm[module_id] | int | 否 |权限| { "module_id1": 4, "module_id2": 7, ...} |

  > ##### 说明
  ```
  jquery $.get 传入参数:
  {
    "perm": {
      "1": 4,
      "2": 7,
      "3": 15
    }
  }
  对应的URL: perm[1]=4&perm[2]=7&perm[3]=15
  角色对模块ID为1的权限为只读(4), 对模块ID为2的权限为修改(7), 对模块ID为2的权限为完全(15)
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 4,
      "name": "商城系统管理员",
      "summary": "",
      "user_count": "0",
      "created_by": "test@smm.cn",
      "created_time": "2016-06-22 18:26:15",
      "perm": null
    },
    "msg": "成功"
  }
  ```

* #### 删除角色
  ```
  POST /admin/role/del
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 角色ID ||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "msg": "成功",
    "data": null
  }
  ```

* #### 获取应用列表
  ```
  GET /admin/app/list
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | field | enum{"name", "title"} | 否 | 查询字段 | |
  | value | string | 否 | 查询值 | |
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |
  | with_module | enum{0, 1} | 否 | 是否返回模块 | 0: 不返回模块, 1: 返回模块 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": [
      {
        "id": 1,
        "name": "admin",
        "title": "管理控制中心",
        "order": 1,
        "module_count": "4",
        "is_active": 1,
        "module": [
          {
            "id": 1,
            "app_id": 1,
            "name": "user_manage",
            "title": "用户管理",
            "order": 1,
            "is_active": 1
          },
          ...
        ]
      },
      ...
    ],
    "msg": "成功"
  }
  ```

* #### 获取应用信息
  ```
  GET /admin/app/info
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 应用ID | |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 1,
      "name": "admin",
      "title": "管理控制中心",
      "order": 1,
      "module_count": "4",
      "is_active": 1,
      "module": null
    },
    "msg": "成功"
  }
  ```

* #### 添加应用
  ```
  POST /admin/app/add
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | name |string|是|名称||
  | title | string |是|显示名称||
  | order | int |否|显示顺序||

  > ##### 请求示例
  ```
  /admin/app/add?name=mall&title=商城系统&order=4
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 4,
      "name": "mall",
      "title": "商城系统",
      "order": 4,
      "module_count": "0",
      "is_active": 1,
      "module": null
    },
    "msg": "成功"
  }
  ```

* #### 更新应用(开启/关闭应用、修改应用名称、修改应用显示名称、修改应用显示顺序)
  ```
  POST /admin/app/update
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 应用ID | |
  | name |string|否|名称||
  | title | string |否|显示名称||
  | order | int |否|显示顺序||
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 4,
      "name": "mall",
      "title": "商城系统",
      "order": 4,
      "module_count": "0",
      "is_active": 0,
      "module": null
    },
    "msg": "成功"
  }
  ```

* #### 删除应用
  ```
  POST /admin/app/del
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 应用ID ||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": null,
    "msg": "成功"
  }
  ```

* #### 获取模块列表
  ```
  GET /admin/module/list
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | field | enum{"name", "title"} | 否 | 查询字段 | |
  | value | string | 否 | 查询值 | |
  | app_id | int | 否 | 应用ID | |
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": [
      {
        "id": 1,
        "app_id": 1,
        "name": "user_manage",
        "title": "用户管理",
        "order": 1,
        "is_active": 1
      },
      {
        "id": 2,
        "app_id": 1,
        "name": "role_manage",
        "title": "角色管理",
        "order": 2,
        "is_active": 1
      },
      ...
    ],
    "msg": "成功"
  }
  ```

* #### 获取模块信息
  ```
  GET /admin/module/info
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 模块ID | |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 1,
      "app_id": 1,
      "name": "user_manage",
      "title": "用户管理",
      "order": 1,
      "is_active": 1
    },
    "msg": "成功"
  }
  ```

* #### 添加模块
  ```
  POST /admin/module/add
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | app_id | int | 是 | 应用ID | |
  | name |string|是|名称||
  | title | string |是|显示名称||
  | order | int |否|显示顺序||

  > ##### 请求示例
  ```
  /admin/module/add?app_id=4&name=bid_review&title=卖盘审核&order=1
  ```

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 18,
      "app_id": 4,
      "name": "bid_review",
      "title": "卖盘审核",
      "order": 1,
      "is_active": 1
    },
    "msg": "成功"
  }
  ```

* #### 更新模块(开启/关闭应用、修改模块名称、修改模块显示名称、修改模块显示顺序)
  ```
  POST /admin/module/update
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 模块ID | |
  | name |string|否|名称||
  | title | string |否|显示名称||
  | order | int |否|显示顺序||
  | is_active | enum{0, 1} | 否 | 状态 | 0: 关闭, 1: 开启 |

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": {
      "id": 18,
      "app_id": 4,
      "name": "bid_manage",
      "title": "卖盘管理",
      "order": 1,
      "is_active": 1
    },
    "msg": "成功"
  }
  ```

* #### 删除模块
  ```
  POST /admin/module/del
  ```

  > ##### 输入参数
  |参数|类型|必填|说明|备注|
  |----|----|----|----|----|
  | auth_token |string|是|||
  | id | int | 是 | 模块ID ||

  > ##### 返回值
  ```json
  {
    "code": 0,
    "data": null,
    "msg": "成功"
  }
  ```


## 模块定义

### 支付系统(pay)

* 账户审核 (account_review)
* 交易审核 (trade_review)
* 出金审核 (withdraw_review)
* 绑定账户审核 (binding_account_review)
* 平台出金 (platform_withdraw)
* 交易管理 (trade_manage)
* 退款管理 (refund_manage)
* 导常处理 (exception_handle)
* 资金数据 (funds_data)
* 系统管理 (system_manage)

### 企业帐号(company)

* 审核企业 (company_review)
* 企业管理 (company_manage)
* 系统管理 (system_manage)

## 附录

### 错误代码说明
|错误代码|说明|
|----|----|
|0|成功
|10001|系统错误
|10002|参数错误
|10003|用户名不合法
|10004|用户不存在
|10005|用户被禁用
|10006|密码错误
|10007|token无效
|10008|权限不足
|10009|新旧密码相同
|10010|用户已存在
|10011|角色不存在
|10012|角色已存在
|10013|应用已存在
|10014|模块已存在
|10015|应用未关闭
|10016|模块未关闭
|10017|应用不存在
|10018|模块不存在
