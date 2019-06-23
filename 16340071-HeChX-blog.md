# Vue+axios+promise 开发过程中的坑
这次的项目前端用的是Vue+axios+promise，因此在这里记录一下在开发过程中踩的坑。  

## Promise
关于Promise的介绍这里就不详细的说了，具体可参考[Promise](https://www.liaoxuefeng.com/wiki/1022910821149312/1023024413276544)  

## Promise先串行后并行
在进入小组详情的界面时，因为要从多个不同api中获取小组的基本信息、小组成员列表等数据，所以需要并行发送axios请求，因此一开始的解决方案是用Promise.all()，然后在获取到所有数据之后统一给data里的变量赋值然后渲染：  
```
this.getGroupDetail(this.team_id)
    .then((data) => {
        this.render(data)
    })
    .catch((err) => {
        console.log(err)
    })

...

render(value) {
    if (value[0].data.code == 200 && value[0].data.data.length != 0) {
        this.group = value[0].data.data[0];
    }
    if (value[1].data.code == 200) {
        this.organizationsList = value[1].data.data;
    }
    if (value[2].data.code == 200) {
        this.isLeader = true;
    } else if (value[2].data.code == 212) {
        this.isLeader = false;
    }
    if (value[3].data.code == 200) {
        this.taskList = value[3].data.data;
    }
},

getGroupDetail(team_id) {
    let p1 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/team/Id?team_id=' + team_id + '&type=0')
            .then((res) => {
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p2 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/user/getCanPublishTasksOrg?teamId=' + team_id)
            .then((res) => {
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })                
    });

    let p3 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/team/Leader?team_id=' + team_id + '&leader=' + this.userInfo.username)
            .then((res) => {
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p4 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/task?type=all&range=' + team_id + '&username=' + this.userInfo.username)
            .then((res) => {
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p = Promise.all([p1, p2, p3, p4]);

    return p;
},
```  
在这里，Promise.all()在所有任务执行完之后会将所有的resolve的数据以数组的形式返回，然后传到render函数中，render函数在进行赋值渲染；  
![返回的数组](image1.png)  

但是后面因为考虑到用户没有登录而直接修改url进入到该页面或者已经登录但是输入url进入到不存在的小组的详情页面等用例，所以打算在获取小组数据前先判断一下用户是否登录已经小组是否存在，确认用户已登录并且用户在该小组中之后再请求小组的信息。因此任务的执行顺序就是先串行再并行，用Promise来写的话就是：  
```
this.judge(this.team_id)
    .then((data) => {
        this.getGroupDetail(data)
    })
    .then((data) => {
        this.render(data)
    })
    .catch((err) => {
        console.log(err)
    })

...

getGroupDetail(team_id) {
    ...     //与上面一致
}

render() {
    ...     //与上面一致
}

judge(team_id) {
    let p = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/team/Member?team_id=' + team_id)
            .then((res) => {
                if (res.data.code == 200) {
                    resolve(team_id);
                } else if (res.data.code == 213) {
                    reject(213);
                } else if (res.data.code == 211) {
                    reject(211);
                }
            })
            .catch((err) => {
                reject(err);
            })
    })

    return p;
},
```
但是奇怪的问题就出现了，这时候传入render函数的参数为undefined  
![](image2.png)  
这就导致了页面的数据都没渲染出来。  

## 解决方法
因为之前先并行再串行的时候返回的数据没问题，而现在在并行前面加了串行之后就出现这种情况，我个人认为应该是先串行再并行的问题，但是在网上找了很多关于Promise的教程或博客都没有提到这种情况，基本上都是只提到并行的用法，或者是并行之后再串行的用法，而没有先串行再并行的情况；因此到目前为止完全了解出现这种情况的原因以及解决方法。  
最后的解决方法只能是放弃统一赋值渲染的render函数，变成在getGroupDetail函数中的每一个axios请求的回调里就把数据赋值给data里的变量。而因为并行之后resolve的数据在后面获取不到，所以后面还有一些串行的请求也不能加在Promise的任务链后面，只能是在axios的回调里嵌套请求。最后的代码变成：  
```
this.judge(this.team_id)
    .then((data) => {
        this.getGroupDetail(data)
    })
    .catch((err) => {
        ...
    })

getGroupDetail(team_id) {
    let p1 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/team/Id?team_id=' + team_id + '&type=0')
            .then((res) => {
                if (res.data.code == 200 && res.data.data.length != 0) {
                    let param = {};
                    param['members'] = [];
                    for (let i = 0, len = res.data.data[0].members.length; i < len; i++) {
                        param['members'].push({username: res.data.data[0].members[i]['member_username']});
                    }
                    this.$axios.post('/api/v1/user/getteammembersavatat', param)
                        .then((response) => {
                            if (response.data.code == 200) {
                                res.data.data[0].members = [];
                                for (let i = 0, len = response.data.data.length; i < len; i++) {
                                    res.data.data[0].members.push(response.data.data[i]);
                                }
                                this.$axios.get('/api/v1/user/getUserBlacklist')
                                    .then((response2) => {
                                        if (response2.data.code == 200 && response2.data.data.length != 0) {
                                            for (let i = 0, len = res.data.data[0].members.length; i < len; i++) {
                                                for (let j = 0, len2 = response2.data.data.length; j < len2; j++) {
                                                    this.$forceUpdate();
                                                    if (res.data.data[0].members[i]['username'] == response2.data.data[j]['username']) {
                                                        res.data.data[0].members[i].inBlacklist = true;
                                                        break;
                                                    } else {
                                                        res.data.data[0].members[i].inBlacklist = false;
                                                    }
                                                }
                                            }
                                        }
                                    })
                                    .catch((error2) => {
                                        console.log(error2);
                                    })
                                
                                this.group = res.data.data[0];
                                resolve(res);
                            } else {
                                reject(response);
                            }
                        })
                        .catch((error) => {
                            reject(error);
                        })
                } else {
                    reject(res);
                }
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p2 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/user/getCanPublishTasksOrg?teamId=' + team_id)
            .then((res) => {
                this.organizationsList = res.data.data;
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
        
    });

    let p3 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/team/Leader?team_id=' + team_id + '&leader=' + this.userInfo.username)
            .then((res) => {
                if (res.data.code == 200) {
                    this.isLeader = true;
                } else if (res.data.code == 212) {
                    this.isLeader = false;
                }
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p4 = new Promise((resolve, reject) => {
        this.$axios.get('/api/v1/task?type=all&range=' + team_id + '&username=' + this.userInfo.username)
            .then((res) => {
                this.taskList = res.data.data;
                resolve(res);
            })
            .catch((err) => {
                reject(err);
            })
    });

    let p = Promise.all([p1, p2, p3, p4]);

    return p;
},
```