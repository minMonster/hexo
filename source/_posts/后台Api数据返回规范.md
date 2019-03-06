---
title: 后台Api数据返回规范
date: 2018-12-17 18:00:00
description: 后台Api数据返回规范.md
tags: [后台]
categories: 后台
---

# 返回码
一般以 http response status code 作为返回码，参考 [rfc7231](https://tools.ietf.org/html/rfc7231)

### 请求成功
- 200 OK：请求成功
- 201 CREATED：成功处理请求，并且有新的资源被创建
- 202 Accepted：表示一个请求已经进入后台排队（异步任务）
- 204 NO CONTENT：成功处理请求，但没有任何数据返回

### 请求失败
- 400 INVALID REQUEST：用户发出的请求有错误，服务器没有进行处理
- 401 Unauthorized：表示用户没有权限（令牌、用户名、密码错误）
- 403 Forbidden：表示用户得到授权（与401错误相对），但是访问是被禁止的
- 404 NOT FOUND：找不到资源，因为资源不存在和路由不存在会有二义性，所以如果是资源不存在，我们当成业务逻辑错误来处理。比如用户不存在，会返回 499，并且会返回一个 error json object
- 422 Unprocesable Entity：请求的参数发生一个验证错误
- 499 业务逻辑错误，说明见下面
- 500 INTERNAL SERVER ERROR：服务器发生错误，用户将无法判断发出的请求是否成功
#### 请求失败返回的数据格式
```json
{
    "message": "手机 不存在。",
    "status_code": 422
}
```

### 业务逻辑错误
**如果是业务逻辑错误，http response status code 会返回 499，同时会有一个 error json object 返回**
```json
{
    "error": {
        "code": 2,
        "user_message": "暂未开放账号注册",
        "internal_message": "暂未开放账号注册",
        "more_info": null
    }
}
```
code: 不同的场景返回不同的值，客户端可根据这个值做不同的业务逻辑处理
user_message: 显示给用户看的错误信息（一般可以弹一个 toast），为空字符串时，不显示

**特殊情况：如果给阿里云 OSS 的回调返回 4xx 错误时，阿里云会给客户端返回错误，客户端拿不到 error 信息。所以阿里云 OSS 的回调需要特殊处理，后台会返回 299 。**


# 数据格式
## 对象
比如获取医生详情
```json
{
    "id": 1,
    "hospital": "速眠睡眠中心",
    "department": "睡眠科",
    "title": "主任医师",
    "qr_code_raw": "",
    "introduction": "速眠睡眠中心主任医生。主要临床专业特长： 各种精神与心理障碍诊疗，尤其是难治性焦虑、抑郁和睡眠障碍的临床诊断和药物治疗。",
    "name": "速眠医生",
    "avatar": "https://sleep-doctor.oss-cn-shenzhen.aliyuncs.com/doctors/avatar/1/93cb9585-fcf3-4888-aed8-678d3a873232.jpg",
    "introduction_no_tag": "速眠睡眠中心主任医生。主要临床专业特长： 各种精神与心理障碍诊疗，尤其是难治性焦虑、抑郁和睡眠障碍的临床诊断和药物治疗。"
}
```

## 分页列表
```json
{
    "data": [
        {
            "id": "8c50ea3a-265c-4d64-b181-03659ffe78e6",
            "type": "App\\Notifications\\ScaleDistribution",
            "read_at": 1527638152,
            "created_at": 1527603221
        },
        {
            "id": "38cfa914-78e0-447a-be19-da5f03d5aa56",
            "type": "App\\Notifications\\ScaleDistribution",
            "read_at": 1527638148,
            "created_at": 1527603114
        }
    ],
    "meta": {
        "pagination": {
            "total": 2,
            "count": 2,
            "per_page": 15,
            "current_page": 1,
            "total_pages": 1,
            "links": {
                "previous": null,
                "next": null
            }
        }
    }
}
```
data: 数据列表，数组类型
meta: 元信息
meta.pagination 分页信息
meta.pagination.total 总数量
meta.pagination.count 当前页数量
meta.pagination.per_page 每页数量
meta.pagination.current_page 当前页码
meta.pagination.total_pages 总页数
meta.pagination.links 链接
meta.pagination.links.next 下一页的链接

# include 使用说明

如果 api 文档有写可以使用 include 参数，如下：

![图片描述](/tfl/captures/2018-06/tapd_21177101_base64_1529562968_14.png)

则可以根据需要来设置 include 的值。

## 例子
### /advisories/26
**只有 advisory 的数据**
```json
{
    "id": 26,
    "status": 4,
    "package_id": 13,
    "order_id": 191,
    "user_id": 2002,
    "doctor_id": 1,
    "start_at": 1526526520,
    "end_at": 1526612920,
    "created_at": 1526526501,
    "updated_at": 1526612920,
    "description": "咯过哦了也在咯做最住\r\n莫咯咯哦了也扣我\r\n李金龙\r\n来PK困咯\r\nKKK女弄\r\n萝莉控\r\n你咯搜狗",
    "service_id": 3,
    "remind_description": "服务已关闭，款项会退回原账户",
    "last_count": 2
}
```

### /advisories/26?include=user
**有 advisory 和 user 的数据**
```json
{
    "id": 26,
    "status": 4,
    "package_id": 13,
    "order_id": 191,
    "user_id": 2002,
    "doctor_id": 1,
    "start_at": 1526526520,
    "end_at": 1526612920,
    "created_at": 1526526501,
    "updated_at": 1526612920,
    "description": "咯过哦了也在咯做最住\r\n莫咯咯哦了也扣我\r\n李金龙\r\n来PK困咯\r\nKKK女弄\r\n萝莉控\r\n你咯搜狗",
    "service_id": 3,
    "remind_description": "服务已关闭，款项会退回原账户",
    "last_count": 2,
    "user": {
        "id": 2002,
        "mobile": "15118132143",
        "nickname": "LDL",
        "name": "李德林",
        "avatar": "https://sleep-doctor.oss-cn-shenzhen.aliyuncs.com/avatar/3/8277ef5a-d0f7-4467-85d5-acb37574cbe2.jpg",
        "area": "未设置",
        "monitor_sn": "",
        "sleeper_sn": "",
        "career": "",
        "created_at": 1517829357,
        "updated_at": 1528094307
    }
}
```

### /advisories/26?include=user,doctor
**有 advisory，user 和 doctor 的数据**
```json
{
    "id": 26,
    "status": 4,
    "package_id": 13,
    "order_id": 191,
    "user_id": 2002,
    "doctor_id": 1,
    "start_at": 1526526520,
    "end_at": 1526612920,
    "created_at": 1526526501,
    "updated_at": 1526612920,
    "description": "咯过哦了也在咯做最住\r\n莫咯咯哦了也扣我\r\n李金龙\r\n来PK困咯\r\nKKK女弄\r\n萝莉控\r\n你咯搜狗",
    "service_id": 3,
    "remind_description": "服务已关闭，款项会退回原账户",
    "last_count": 2,
    "user": {
        "id": 2002,
        "mobile": "15118132143",
        "nickname": "LDL",
        "name": "李德林",
        "avatar": "https://sleep-doctor.oss-cn-shenzhen.aliyuncs.com/avatar/3/8277ef5a-d0f7-4467-85d5-acb37574cbe2.jpg",
        "area": "未设置",
        "monitor_sn": "",
        "sleeper_sn": "",
        "career": "",
        "created_at": 1517829357,
        "updated_at": 1528094307
    },
    "doctor": {
        "id": 1,
        "hospital": "速眠睡眠中心",
        "department": "睡眠科",
        "title": "主任医师",
        "qr_code_raw": "",
        "introduction": "速眠睡眠中心主任医生。主要临床专业特长： 各种精神与心理障碍诊疗，尤其是难治性焦虑、抑郁和睡眠障碍的临床诊断和药物治疗。",
        "name": "速眠医生",
        "avatar": "https://sleep-doctor.oss-cn-shenzhen.aliyuncs.com/doctors/avatar/1/93cb9585-fcf3-4888-aed8-678d3a873232.jpg",
        "introduction_no_tag": "速眠睡眠中心主任医生。主要临床专业特长： 各种精神与心理障碍诊疗，尤其是难治性焦虑、抑郁和睡眠障碍的临床诊断和药物治疗。"
    }
}
```

# token 自动刷新机制

token 如果已经过期，但在可刷新有效期内，服务器会自动生成一个新的 token，并且在当前请求的 response header 返回，
格式为：
```
Authorization: Bearer {newToken}
```
客户端要检查每个请求的 response header 是否有 Authorization，如果有，要把 {newToken} 保存下来并用新的 token 来请求。
重新生成 token 后，旧 token 还可以用 60 秒，所以客户端不用处理并发的问题。


