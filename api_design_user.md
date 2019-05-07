## 用户注册

``register(infoForm)``：

+ 将用户在表单中填写的个人信息提交至服务器进行注册

+ POST: `/api/user/create`
  
    +  Content-Type: multipart/form-data
    + 表单数据包含个人信息的文本数据，以及头像，认证照片的图像文件
    + 提交的表单数据在服务端经过formidable模块解析后的格式为：
    
    ```json
    {
        "fields": {
            "type": "0",
            "username": "hzh",
            "password": "123",
            "name": "hzh",
            "school": "SYSU",
            "grade": "3",
            "phone": "",
            "weChat": "",
            "qq": ""
        },
        "files": { 
            "authImg": {
                "size": 12345,
                "path": "",
                "name": "",
                "type": "image/jpeg"
            },
            "avatar": {
                "size": 12345,
                "path": "",
                "name": "",
                "type": "image/jpeg"
            },
        } 
    }
    ```
    注：若未上传头像或认证照片，则``files``中无``avatar``或``authImg``键值对。
    + 返回信息

        + 200 创建成功,并返回创建成功的用户信息
        ```json
        {
            "code": 200,
            "msg": "success",
            "data": {
                "username": ""
            }
        }
        ```
        + 409 用户名已存在
        ```json
        {
            "code": 409,
            "msg": "该用户已存在",
            "data": null
        }
        ```
        + 500 服务器异常

        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": null
        }
        ```

## 用户登录

``login(type, username, password)``: 根据登录类型，用户名， 密码进行登录，服务器给创建一个session并把session-id作为cookie发送给客户端。

+ POST: `/api/user/login`
  
    + Content-Type:  application/json
    + 上传数据格式
    
    ```json
    {
        "type":0,
        "username":"hzh",
        "password":"123"
    }
    ```
    + 返回信息
        + 200 登录成功
        ```json
        {
            "code": 200,
            "msg": "success",
            "data": null
        }
        ```
        + 412 用户名或密码错误
        ```json
        {
           "code": 412,
            "msg": "用户名或密码错误",
            "data": null
        }
        ```
        + 413 账户类型错误
        ```json
        {
           "code": 413,
            "msg": "账户类型错误",
            "data": null
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": null
        }
        ```

## 用户退出

``logout()``: 服务器通过客户端请求的cookie头找到对应session，改变session状态。

## 获取用户信息

``getUserInfo(username)``: 通过用户名获取用户信息。

+ GET: `/api/user/getuser?username={username}`
  
    + 返回信息
        + 200 查询成功并返回用户基本信息
        ```json
        {
            "code": 200,
            "msg": "success",
            "data": {
                "username": "hzh",
                "name": "hzh",
                "school": "SYSU",
                "grade": "3",
                "phone": "",
                "weChat": "",
                "qq": ""
            }
        }
        ```
        + 400 参数错误
        ```json
        {
            "code": 400,
            "msg": "Wrong query param.",
            "data": null
        }
        ```
        + 412 查询用户不存在
        ```json 用户不存在
        {
            "code": 412,
            "msg": "用户不存在",
            "data": null
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": null
        }
        ```

## 获取用户头像

+ 用户头像作为静态资源

+ GET: `/:path/avatar_{username}`
  

## 获取用户接受的已完成的任务
+ GET: `/api/user/getAcceptedFinishedTasks?username={username}`
    + 返回用户接受的已完成的任务列表
    ```json
    {
        "code": 200,
        "msg": "success",
        "data": [
            {   
                "taskId": 1,
                "title": "问卷调查",
                "introduction": "",
                "starttime": "2019-05-01 00:00:00",
                "endtime": "2019-05-02 00:00:00",
                "score": 3.5,
                "money": 0.1
            },
            {   
                "taskId": 2,
                "title": "取快递",
                "introduction": "",
                "starttime": "2019-05-02 00:00:00",
                "endtime": "2019-05-03 00:00:00",
                "score": 4.0,
                "money": 12.0
            }
        ]  
    }
    ```
## 获取用户发布的已完成的任务
+ GET: `/api/user/getPublishedFinishedTasks?username={username}`
    + 返回用户发布的已完成的任务列表
    ```json
    {
        "code": 200,
        "msg": "success",
        "data": [
            {   
                "taskId": 1,
                "title": "问卷调查",
                "introduction": "",
                "starttime": "2019-05-01 00:00:00",
                "endtime": "2019-05-02 00:00:00",
                "score": 3.5,
                "money": 0.1
            },
            {   
                "taskId": 2,
                "title": "取快递",
                "introduction": "",
                "starttime": "2019-05-02 00:00:00",
                "endtime": "2019-05-03 00:00:00",
                "score": 4.0,
                "money": 12.0
            }
        ]  
    }
    ```
## 获取用户发布的等待接受的任务
+ GET: `/api/user/getPublishedWaitedTasks?username={username}`
    + 返回用户发布的已完成的任务列表
    ```json
    {
        "code": 200,
        "msg": "success",
        "data": [
            {   
                "taskId": 1,
                "title": "问卷调查",
                "introduction": "",
                "starttime": "2019-05-01 00:00:00",
                "endtime": "2019-05-02 00:00:00",
                "score": 3.5,
                "money": 0.1
            },
            {   
                "taskId": 2,
                "title": "取快递",
                "introduction": "",
                "starttime": "2019-05-02 00:00:00",
                "endtime": "2019-05-03 00:00:00",
                "score": 4.0,
                "money": 12.0
            }
        ]  
    }
    ```
## 任务发布者给完成此任务的接收者评分

+ `setRate(taskId, value)`: 服务端通过cookie判断用户是否为`taskId`对应的任务的发布者，若是，则将发布者对完成此任务的接受者的评分设为`value`

+ POST: `/api/user/setRate`
  
    + Content-Type:  application/json
    + 上传数据格式
    ```json
    {
        "taskId": 1,
        "value": 3.5
    }
    ```
    + 返回信息
      + 200 更新评分，返回新的评分值
      ```json
      {
          "code": 200,
          "msg": "success",
          "data": 3.5
      }
      ```
      + 401 无权限，返回原评分值
      ```json
      {
          "code": 401,
          "msg": "failed",
          "data": 2.5
      }
      ```
       + 400 未找到任务
      ```json
      {
          "code": 400,
          "msg": "failed",
          "data": null
      }
      ```
## 获取组织信息

+ GET: `/api/user/getorg?username={username}`
    + 返回信息
        + 200 查询成功并返回用户除密码外的其他信息
        ```json
        {
            "code": 200,
            "msg": "success",
            "data": {
                "username": "test4",
                "score": 100,
                "money": 100,
                "true_name": "test4",
                "school_name": "SYSU",
                "grade": 4,
                "avatar": null,
                "nickname": "test4",
                "wechat": null,
                "QQ": null,
                "phone_number": null,
                "account_state": 1
            }
        }
        ```
        + 400 参数错误
        ```json
        {
            "code": 400,
            "msg": "Wrong query param.",
            "data": null
        }
        ```
        + 412 查询组织不存在
        ```json
        {
            "code": 412,
            "msg": "用户不存在",
            "data": {}
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": {}
        }
        ```

## 更新用户信息

``updateUserInfo(info)``: 提供要修改后的数据，info为一个对象。通过cookie验证身份。
+ PUT:`/api/user/update`
    + 提交数据格式
    ```json
    {
        "username": "hyx",
        "password":"123",
        "score": 0,
        "money": 1632,
        "true_name": "韩",
        "school_name": "SYSU",
        "grade": 5,
        "avatar": "default url waiting be set",
        "nickname": "hyx",
        "wechat": null,
        "QQ": null,
        "phone_number": null,
        "account_state": 0
    }
    ```
        + 用户名用户不得更改，但需从前端将用户名传到后端，一遍后端进行数据库查找
    + 返回信息
        + 200 修改成功并返回修改后的用户信息
        ```json
        {
            "code": 200,
            "msg": "更新成功",
            "data": {
                "username": "hyx",
                "score": 0,
                "money": 1632,
                "true_name": "韩",
                "school_name": "SYSU",
                "grade": 5,
                "avatar": "default url waiting be set",
                "nickname": "hyx",
                "wechat": null,
                "QQ": null,
                "phone_number": null,
                "account_state": 0
            }
        }
        ```
        + 412 更新失败
        ```json
        {
            "code": 412,
            "msg": "更新失败",
            "data": "error"
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": {}
        }
        ```
## 充值

``deposit()``
+ POST: `/api/user/recharge`
    + 提交数据格式
    ```json
    {
        "username": "hyx",
        "money": 1632
    }
    ```
    + 返回数据格式
        + 200 充值成功并返回用户当前余额
        ```json
        {
            "code": 200,
            "msg": "充值成功",
            "data": 6528
        }
        ```
        + 412 用户账户余额不足
        {
            "code": 412,
            "msg": "账户余额不足",
            "data": "error"
        }
        + 413 充值用户不存在 
        ```json
        {
            "code": 413,
            "msg": "无当前用户",
            "data": "error"
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": {}
        }
        ```

## 修改信用分数
+ POST: `/api/user/score`
    + 提交数据格式
    ```json
    {
            "username": "hyx",
        "score": 10
    }
    ```
    + 返回数据格式
        + 200 改变用户信用度并返回用户当前信用度
        ```json
        {
            "code": 200,
            "msg": "增加信用度成功",
            "data": 20
        }
        ```
        + 412 扣除用户信用度时用户账户信用度不足
        {
            "code": 412,
            "msg": "账户信用分数不足",
            "data": "error"
        }
        + 413 改变信用度的用户不存在 
        ```json
        {
            "code": 413,
            "msg": "无当前用户",
            "data": "error"
        }
        ```
        + 500 服务器异常
        ```json
        {
            "code": 500,
            "msg": "failed",
            "data": {}
        }
        ```
## 提现

``withdraw()``

+ 第一次迭代暂无此功能