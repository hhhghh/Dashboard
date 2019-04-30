

# 搜索任务界面

**注意，所有URL都是以我们的域名`/api`开始的，比如域名为`hhhghh.com`，那么就是`hhhghh.com/api/task?...` ，`hhhghh.com/task/`之类的这种域名留给前端的正常页面展示**

1. getGroup(user.id)
  根据user.id查找用户所有加入的小组group信息
  （在筛选列表中，显示用户所加入的小组名）

  > `GET: /team?member_username={:username}` 
  > 查看 team API 部分的实现


2. getTask(type,range,user.id)
  type:任务类型,range:任务发布范围`all`或者`group.id`
  根据type，range查找所有任务Tasks，
  （根据user.id从`屏蔽列表`找到屏蔽该user的releaser查找releaser所有发布的任务t_set0.
  根据Tasks信息，比较每个task.score与user.score，
  若task.score > user.score, t_set1.append(task)
  返回(Tasks - t_set0 - t_set1)
  
  > **OK**
  > `GET: /task?type={:type}&range={:range}&username={:username}`
  >
  > 预计实现时几个参数可以自由组合
  > 成功返回200，失败返回412


# 任务发布界面
3. `getUserInfo()`
  返回用户信息
  (为了比较发布任务的钱与用户的钱的大小)
 
  > `GET: /user?username={}`
  >

3. releaseTask(task)  
  task信息：
  title
  introduce
  number
  money       
  user.id
  score
  endtime
  starttime
  content(任务具体描述，或联系方式等)
  type（问卷调查选择上传yaml格式文件）
  range(任务范围){'all','group1.id','group2.id'}
  。。。。
  数据库添加task
  返回任务发布成功

  > **OK**
  > `POST: /task/` 
  >
  > 使用`POST`提交数据，以JSON格式
  > 
  > 成功返回 200 与插入的数据
  > 失败返回 412

# 我的接受任务界面
5. getAcceptTask(type, range,state, user.id)
  type: 任务类型, range:任务发布范围,state:任务状态`全部, 已完成, 正在做，待审核`
  根据 type, range,state,user.id 查找用户接受的任务
  返回用户接受的任务信息

  > **OK**
  > `GET: /task?acceptable=true&type={type}&range={range}&state={state}&username={username}`
  >
  > 使用`acceptable=true`做限制？待定待学习

# 我的发布任务界面
getGroup(user.id)

6. getReleaseTask(type, range,state,user.id)
  type: 任务类型, range:任务发布范围，state:任务状态`全部, 已完成, 正在做，待审核`
  根据 type, range,state,user.id 查找用户发布的任务
  返回用户发布的任务信息

  > **OK**
  > `GET: /task?release_user={username}`
  > 
  > 推荐使用 publisher 和数据库保持一致，用 release_user 也没关系


# 任务详情页面
7. getTask(task.id, user.id)
  根据task.id查看任务
  返回任务信息
  希望并返回user与该任务的联系（发布，接受，或未接受）

  > **OK**
  > 用两个API来完成吧，发起两次请求可以么？
  >
  > `GET: /task?task_id={}`
  >
  > `GET: /task/relations?task_id={id}&username={username}`
  > 
  > 同样，成功 200，并返回插入的数据，失败 412，

## 发布人的任务详情界面
8. getTaskRelations(task.id)
  根据 task.id 查找 任务联系tr(task relation)
  task relation信息
  user.id 接受人id
  state   任务状态

  返回任务联系信息
  
  > **OK**
  > `GET: /task/releations?task_id={}`
  >
  > 成功200，返回一个列表，所有接受人的信息
  > 失败412


9. getUserInfo(user.id)
  返回头像URL
  （或输入一组user.id返回一组user.url  $$不清楚)
  ![](/image/api_images/1.PNG)
  ![](/image/api_images/2.PNG)

  > `GET: /user/avater?username={username}`
  >
  > 或者直接返回所有用户信息可以么，不单独返回头像了就


10. confirmTaskComplement(user.id, task.id, score)
  根据user.id和task.id查找任务联系tr
  修改tr的任务状态state
  以及根据情况修改task.state
  score:发布人对完成任务者的评价分数
  修改完成任务者user.score
  返回成功信息

  > `PATCH: /task/`
  >
  > 不太清楚`PATCH`调用的方法，大概是和`POST`差不多的，提交一个`JSON`的数据对象，进行局部更新


10. cancelTask(task.id)
  发布人取消任务 $$取消任务的条件。。不清楚
  删除任务
  返回成功信息

  > **OK**
  > `DELETE: /task`
  >
  > 发起请求，取消任务
  >
  > 删除任务，成功200，失败412
  > 
  > TODO: 
  > 
  > 条件检查 

## 未接受该任务的任务详情界面
12. acceptTask(task.id, user.id)
   新建任务联系tr
   返回成功信息

  > **OK**
  > `POST: /task/relations`
  > 
  > 接受任务，在 relations 中添加一行
  > 成功200，失败412，可能会是由于已接受而不可再次接受

## 接受了该任务的任务详情界面
13. getTaskRelation(user.id)
   返回任务联系tr
   用来显示任务状态
   
   > **OK**
   > `GET: /task/releations?username={}`
   > 
   > 根据用户来查，成功200，失败412

14. quitTask(task.id, user.id)
   根据task.id， user.id删除对应任务联系tr
  
   > **OK**
   > `DELETE：/task/relations` 
   > 成功200，失败412




