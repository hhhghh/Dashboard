## 用户注册

``register(type, info)``：

- type参数为注册类型，0表示普通用户，1表示机构；
- info为注册时填写的个人信息，是一个对象，可以通过key取得各信息，如logo,school,grade,username等。

## 用户登录

``login(type, username, password)``: 根据登录类型，用户名， 密码进行登录，服务器给创建一个session并把session-id作为cookie发送给客户端。

## 用户退出

``logout()``: 服务器通过客户端请求的cookie头找到对应session，改变session状态。

## 获取用户信息

``getUserInfo(username)``: 通过用户名获取用户信息。

## 更新用户信息

``updateUserInfo(info)``: 提供要修改后的数据，info为一个对象。通过cookie验证身份。

## 充值

``deposit()``

## 提现

``withdraw()``