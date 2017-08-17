---
title: "Willin: AirX 开发最佳实践"
date: 2017-08-17 08:29:57
categories: Dev
tags: [node.js, airx, dev]
---

# 接入流程

1. 完成认证(个人/组织)
2. 创建应用
3. 完成应用功能的开发测试
4. 提交审核
5. 审核完成后, 正式上线

<!--more-->

# 开发流程

创建应用后, 将得到:

```js
{
  SecretId: 'AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA',
  SecretKey: 'Gu5t9xGARNpq86cd98joQYCN3Cozk1qA'
}
```

这样的信息, 妥善保管 `SecretKey` 切勿在任何情况下进行泄露.

## 公共请求参数

| 名称 | 类型 | 描述 | 是否必选 |
| --: | :--: | :-- | :--: |
| Timestamp | UInt | 当前UNIX时间戳，可记录发起API请求的时间(如:1495608418)。 | √ |
| Nonce | UInt | 随机正整数，与 Timestamp 联合起来, 用于防止重放攻击。 | √ |
| SignatureMethod | String | 用来说明Signature签名使用的签名算法，字段值可为"HmacSHA256"和"HmacSHA1"。目前支持HmacSHA256和HmacSHA1两种签名算法，用户不传该字段默认使用HmacSHA1签名算法。若用户填写的字段值错误，则也默认使用HmacSHA1签名算法。 | × |
| SecretId | String | 在[云API密钥](#)上申请的标识身份的 SecretId，一个 SecretId 对应唯一的 SecretKey , 而 SecretKey 会用来生成请求签名 Signature。具体可参考 [签名方法](#) 页面。 | √ |
| Signature | String | 请求签名，用来验证此次请求的合法性，由系统根据输入参数自动生成。具体可参考 [签名方法](#) 页面。 | √ |

假设用户想要查询用户是否存在，则其GET请求链接的形式可能如下:

```
https://api.dwi.ng/user/check/13312341234?
SecretId=xxxxxxx
&Timestamp=1495608418
&Nonce=59485
&Signature=mysignature
&<接口请求参数>
```

如果是POST请求,参数必须全部放置在表单内(而非使用QueryString方式传递)。

### 公共返回结果

请求成功:

```js
{
  "status": 1,
  "data": {
    // 成功返回的结果数据
  }
}
```

请求失败:


```js
{
  "status": 0,
  "code": 1000 // 错误码
}
```


## 签名机制

假设用户的 SecretId 和 SecretKey 分别是：

> SecretId: AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA
> 
> SecretKey: Gu5t9xGARNpq86cd98joQYCN3Cozk1qA

注意：这里只是示例，请用户根据自己实际的SecretId和SecretKey和请求参数进行后续操作！

以查看用户是否存在(/user/check/:username)请求为例，当用户调用这一接口时，其请求参数可能如下:

| 参数名称 | 中文 | 参数值 |
| --: | :-- | :-- |
| SecretId | 密钥Id | AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA |
| Timestamp | 当前时间戳 | 1465185768 |
| Nonce | 随机正整数 | 11886 |
| SignatureMethod | 签名方式 | HmacSHA256 |

### 对参数排序

首先对所有请求参数按参数名做字典序升序排列，所谓字典序升序排列，直观上就如同在字典中排列单词一样排序，按照字母表或数字表里递增顺序的排列次序，即先考虑第一个“字母”，在相同的情况下考虑第二个“字母”，依此类推。您可以借助编程语言中的相关排序函数来实现这一功能，如php中的ksort函数。上述示例参数的排序结果如下:

```js
{
    'Nonce' : 11886,
    'SecretId' : 'AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA',
    'SignatureMethod' : 'HmacSHA256',
    'Timestamp' : 1465185768,
    'mobile': '13300001111',
    'password': 'xxxxxxxxxxxxxxxx',
    'key': '2222',
    'code': '1111',
    'guid': '123456',
    'device': 'iphone'
}
```

使用其它程序设计语言开发时, 可对上面示例中的参数进行排序，得到的结果一致即可。

### 拼接请求字符串

此步骤生成请求字符串。

将把上一步排序好的请求参数格式化成“参数名称”=“参数值”的形式。

**注意：1、“参数值”为原始值而非url编码后的值。2、若输入参数的Key中包含下划线，则需要将其转换为“.”，但是Value中的下划线则不用转换。如Placement_Zone=CN_GUANGZHOU,则需要将其转换成Placement.Zone=CN_GUANGZHOU。**

然后将格式化后的各个参数用"&"拼接在一起，最终生成的请求字符串为:

```
Nonce=33954&SecretId=AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA&Timestamp=1496305987&code=1111&device=iph
one&guid=123456&key=2222&mobile=13300001111&password=xxxxxxxxxxxxxxxx
```

### 拼接签名原文字符串

此步骤生成签名原文字符串。
签名原文字符串由以下几个参数构成:

1) 请求方法: 支持 POST 和 GET 方式, 这里使用 GET 请求, 注意方法为全大写。
2) 请求主机: 请求域名为：api.dwi.ng。
3) 请求路径: 如 /user/check/13112341234。
4) 请求字符串: 即上一步生成的请求字符串。

签名原文串的拼接规则为:

> 请求方法 + 请求主机 + 请求路径 + ? + 请求字符串

示例的拼接结果为：

```
POSTapi.dwi.ng/user/register/mobile?Nonce=33954&SecretId=AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA&Timestamp=1496305987&code=1111&device=iph
one&guid=123456&key=2222&mobile=13300001111&password=xxxxxxxxxxxx
```

### 生成签名串

此步骤生成签名串。

**注意：这里要根据您指定的签名算法（即SignatureMethod参数）生成签名串。当指定SignatureMethod为HmacSHA256时，需要使用HmacSHA256计算签名，其他情况请使用HmacSHA1计算签名。**

首先使用签名算法（HmacSHA256或HmacSHA1）对上一步中获得的签名原文字符串进行签名，然后将生成的签名串使用 Base64 进行编码，即可获得最终的签名串。

具体代码如下，以 PHP/node.js 语言为例，由于本例中所用的签名算法为HmacSHA256，因此生成签名串的代码如下：

```php
// php
$secretKey = 'Gu5t9xGARNpq86cd98joQYCN3Cozk1qA';
$srcStr = 'POSTapi.dwi.ng/user/register/mobile?Nonce=33954&SecretId=AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA&Timestamp=1496305987&code=1111&device=iph';
$signStr = base64_encode(hash_hmac('sha256', $srcStr, $secretKey, true));
echo $signStr;
```

```js
// node.js
const crypto = require('crypto');

const secretKey = 'Gu5t9xGARNpq86cd98joQYCN3Cozk1qA';
const srcStr = 'POSTapi.dwi.ng/user/register/mobile?Nonce=33954&SecretId=AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA&Timestamp=1496305987&code=1111&device=iphone&guid=123456&key=2222&mobile=13300001111&password=xxxxxxxxxxxx';


const hmac = crypto.createHmac('sha256', secretKey);
const signStr = hmac.update(new Buffer(srcStr, 'utf8')).digest('base64');

console.log(signStr);
```

最终得到的签名串类似:

```
ZvlpRKOA6iQD+DGtjuiWUpN/H81v0ChwDtS2syL2hzw=
```

同理，当您指定签名算法为HmacSHA1时（即未指定SignatureMethod参数为HmacSHA256），生成签名串的代码如下：

```js
// node.js
const crypto = require('crypto');

const secretKey = 'Gu5t9xGARNpq86cd98joQYCN3Cozk1qA';
const srcStr = 'POSTapi.dwi.ng/user/register/mobile?Nonce=33954&SecretId=AKIDz8krbsJ5yKBZQpn74WFkmLPx3gnPhESA&Timestamp=1496305987&code=1111&device=iphone&guid=123456&key=2222&mobile=13300001111&password=xxxxxxxxxxxx';


const hmac = crypto.createHmac('sha1', secretKey);
const signStr = hmac.update(new Buffer(srcStr, 'utf8')).digest('base64');

console.log(signStr);
```

最终得到的签名串类似：

```
+QeK/Gvh2nKz0kBrscT/IHJBbYo=
```

使用其它程序设计语言开发时, 可用上面示例中的原文进行签名验证, 得到的签名串与例子中的一致即可。

### 签名串编码

生成的签名串并不能直接作为请求参数，需要对其进行 URL 编码。
如上一步生成的签名串为PSWUwVv4TfW2sQP1FT9KbbLjIZ9bmK8f1cL6fbpf3KI=，则其编码后为PSWUwVv4TfW2sQP1FT9KbbLjIZ9bmK8f1cL6fbpf3KI%3D。因此，最终得到的签名串请求参数(Signature)为：PSWUwVv4TfW2sQP1FT9KbbLjIZ9bmK8f1cL6fbpf3KI%3D，它将用于生成最终的请求URL。

**注意：如果用户的请求方法是GET，则对所有请求参数的参数值均需要做URL编码；此外，部分语言库会自动对URL进行编码，重复编码会导致签名校验失败。**


### 鉴权失败

当鉴权不通过时，可能出现的错误如下表：

| 错误代码 | 错误类型 | 错误描述 |
| :--: | :-- | :-- |
| 4100 | 身份认证失败 | 身份验证失败，请确保您请求参数中的Signature按照上述步骤计算正确，特别注意Signature要做url编码后再发起请求。| 
| 4101 | 未被开发商授权访问本接口 | 该子用户未被授权调用此接口。请联系开发商授权，详情请查阅授权策略。| 
| 4102 | 未被开发商授权访问本接口中所操作的资源 | 请问的资源参数中，存在未被开发商授权授权访问的资源，请在message字段中查看无权查看的资源ID。请联系开发商授权，详情请查阅授权策略。| 
| 4103 |  非开发商的SecretId暂不支持调用本接口 | 子用户的SecretID不支持调用此接口,只有开发商有权调用。| 
| 4104 | secretId不存在 | 签名所用的secretId不存在，也可能是密钥状态有误，请确保API密钥有效且未被禁用。| 
| 4110 | 鉴权失败 | 权限校验失败，请确保您有使用所访问资源的权限。| 
| 4500 | 重放攻击错误 |  请注意 Nonce 参数两次请求不能重复, Timestamp 与大翼服务器相差不能超过 2 小时。| 

## 错误码


```js
{
  UNKOWN_ERROR: 1000, // 未知错误
  INVALID_PARAMS: 1001, // 参数不合法
  DB_ERROR: 1100, // 数据库错误
  AZURE_ERROR: 1101, // 微软基础服务错误

  INVALID_SMS_CODE: 2000, // 短信验证码错误
  LIMITATION_SMS_CODE: 2001, // 短信验证码发送限制
  USERNAME_EXISTED: 2002, // 用户名已经被注册
  USERNAME_BINDED: 2003, // 已绑定用户名
  EMAIL_EXISTED: 2004, // 邮箱已经被注册
  MOBILE_EXISTED: 2005, // 手机号已经被注册
  ORG_EXISTED: 2006, // 组织名称已存在

  INVALID_PASSWORD: 2100, // 密码强度不符合要求
  INVALID_ENCRYPTION: 2101, // 加密错误

  INVALID_USERNAME_OR_PASSWORD: 2200, // 用户不存在或密码错误
  INVALID_LOGIN_DEVICE: 2201, // 未授权的登录设备
  INVALID_USER_DEVICE_OWN: 2202, // 用户未拥有此设备
  INVALID_LOGIN_AUTH: 2203, // 登录超时或非法登录Auth Token
  USER_JOINED_ORG: 2300, // 用户已经加入/创建组织
  ADMIN_NOT_EXIST: 2301, // 组织管理员不存在
  PARENT_ORG_NOT_EXIST: 2302, // 上级组织不存在
  ORG_NOT_EXIST: 2303, // 组织不存在
  UID_NOT_MATCH_OID: 2304, // 用户和组织不匹配
  USER_MOBILE_EXITED: 2305, // 用户手机号尚未注册
  USER_TO_ADD_NOT_EXIST: 2306, // 待添加的用户不存在
  USER_TO_DELETE_NOT_EXIST: 2307, // 待删除的用户不存在
  USER_NOT_JOINED_ORG: 2308, // 用户未加入组织
  USER_PRIVILEGE_NOT_SUFFICIENT: 2309, // 用户权限不足
  ORG_FORBIDDEN: 2310, // 无权操作该组织
  PARENT_ORG_FORBIDDEN: 2311, // 无权操作上级组织

  DEVICE_ALREADY_REGISTERRED: 2500, // 该设备已经注册
  DEVICE_NOT_EXIST: 2501, // 设备未注册
  DEVICE_ENABLE_IOTHUB_FAILURE: 2502, // 设备iothub启用失败
  DEVICE_DISABLE_IOTHUB_FAILURE: 2503, // 设备iothub禁用失败
  TAPE_CREATE_FAILURE: 2601, // 创建录制任务失败
  PLAN_NOT_ENDED: 2602, // 已经申请的任务没有终止
  PLAN_ALREADY_ENDED: 2603, // 已经申请的任务已经终止
  VIDEO_CODE_CHECK_FAILURE: 2604, // 视频直播索引码无效
  DID_AND_STARTTIME_USED: 2605, // 数据库中用户id和起飞时间已占用
  ORG_DATA_ERROR: 2606, // 数据库中oid上级递进查询出现问题，例子：出现子组织a的父组织b的父组织，又是a
  ENDTIME_EARLIER_THAN_STARTTIME: 2607, // 传递参数中，结束时间比开始时间要早，错误
  INVALID_FILE_EXT: 2400, // 文件格式不合法
  INVALID_FILE_TYPE: 2401, // 文件类型不合法
  USER_NOT_OWN_APP_KEY: 2700, // 用户无权处理此app版本
  WRONG_INPUT_PARAMS: 2701, // 用户输入参数有误
  ALREADY_VERIFIED_CAN_NOT_EDITED: 2702, // 已经验证完毕，不能再次编辑
  TARGET_NOT_EXISTED: 2703, // 目标app记录不存在
  APP_VERSION_DELETED: 2704, // APP版本已被删除
  CAN_NOT_FIND_APP_VERSION: 2705, // appkey对应的app找不到可用版本
  DATA_ORGANIZATION_WRONG: 2706, // 组织用户数据错误
  // 签名相关
  AUTHORIZE_FAILED: 4100, // 鉴权失败
  INVALID_APPKEY: 4104, // 不存在的SecretId
  REPLAY_ATTACKS: 4500 // 重放攻击
}
```
