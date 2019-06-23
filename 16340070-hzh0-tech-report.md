# Techblog
|学号|github id|
|---|---|
|16340070|hzh0|

## 关于后端使用koa2框架头像图片的存储和更新

我们的项目采用koa2作为后端的开发框架，我主要负责的是后端的用户部分的功能实现，在用户头像的存储上，我们决定将图片存储在服务器的文件夹下，在数据库中只存储头像路径，这样就能最大限度的提升数据库的性能。关于注册，前端需要传送头像图片和必要的一些其他基本信息，一种设想是通过数据转码，将头像转码后发给后端进行解码再存储。这种方法是将图片的传输和用户其它信息的json文件分开传送。另一种方法是直接通过form-data格式将文件和text信息直接一起传送到后端，然后通过formidable处理form-data类型数据然后进行头像存储和其他数据。

### [formidable](https://www.npmjs.com/package/formidable)对注册用的头像的保存

由于我们注册的时候需要先得到用户的头像和所有信息，然后再存储到数据库中，最后在数据库存储成功后返回给前端一个注册成功的信息。所以注册的功能是在等到接受完所有信息后再进行的，但由于formidable所提供给我们的api并不是异步的，所以我们在编码过程中直接使用官方提供的api会造成数据并未接收完就直接进行接下来的数据库操作而造成注册失败了。所以我们接受数据的时候需要等待，即能够await接收完数据再进行数据库操作，所以我们首先需要将form.parse(),即formidable对数据的处理异步化。我所采用的是Promise将其异步化。
具体代码：
```js
function formidablePromise (req, opts) {
    return new Promise(function (resolve, reject) {
        var form = new formidable.IncomingForm(opts)
        form.keepExtensions = true;     
        form.uploadDir = 'static/uploads/user/';
        form.parse(req, function (err, fields, files) {
            var extname = null
            if (files.avatar) {
                var extname = path.extname(files.avatar.path)
                var oldpath = files.avatar.path
                var newpath = form.uploadDir + fields.username + extname
                var issave = false
                if (!fs.existsSync(newpath)) {
                    fs.rename(oldpath, newpath, function(err) {
                        if (err) {
                            throw err
                        }
                    })
                    issave = true
                }
            }
            if (err) return reject(err)
            resolve({ fields: fields, files: files, newpath: newpath, oldpath: oldpath, extname: extname, issave: issave })
        })
    })
}
```
这样，我们在后端处理数据时，就能直接这样：
```js
var data = await formidablePromise(ctx.req, null);
```
以此来等待formidable.parse处理完数据，然后使用data来处理`resolve({ fields: fields, files: files, newpath: newpath, oldpath: oldpath, extname: extname, issave: issave })`中返回给我们的各项信息了。
在处理过程中，我们需要使用
```js
form.uploadDir = 'static/uploads/user/';
```
来指定图片的存储路径，若没有这句，文件将会自动存储在默认路径下。我存储的这个路径将会存储在该函数所在的js文件所在的文件夹的同级目录static的下级的下级文件夹即user文件夹下。到这一步我们的存储还没有结束，由于直接存储文件名会是生成的无序数字序列，所以我们需要对文件进行改名
```js
var newpath = form.uploadDir + fields.username + extname`
```
我们的新的文件的名字将会是fields字段中的username名加上文件扩展名，存储的路径就是之前存的地方，所以前面的路径不用更改。
在这里，我们还需要考虑到一种情况，就是假设已经有用户`userA`存在，那么其头像路径会是"`....static/uploads/user/userA.jpg`"，现在有一个用户又以用户名`userA`进行注册，由于用户的注册是将文件和用户信息字段一起发送的，所以拿到注册用户名进行用户是否已存在的判断是在存储图片之后进行的，所以以上的状况会变成第二个用户虽然会在之后的判断中因为重复的用户名而造成注册失败，但他的头像文件由于之前存储的原理造成覆盖之前用户头像的情况。虽然路径没变，但实际上已经更改了图片内容，所以在存储时我们需要进行判断
```js
var issave = false
if (!fs.existsSync(newpath)) {
    fs.rename(oldpath, newpath, function(err) {
        if (err) {
            throw err
        }
    })
    issave = true
}
```
只有当用户的头像是首次创建时才存储，即对重复的用户，其在之后是一定不会注册成功的，所以我们可以通过这样的方法来不对其头像进行更名，然后将其存储路径传出，在判断重名后直接通过路径删除图片。
完成的注册代码：
```js
static async register(ctx) {
    function formidablePromise (req, opts) {
        return new Promise(function (resolve, reject) {
            var form = new formidable.IncomingForm(opts)
            form.keepExtensions = true;     
            form.uploadDir = 'static/uploads/user/';
            form.parse(req, function (err, fields, files) {
            var extname = null
            if (files.avatar) {
                var extname = path.extname(files.avatar.path)
                var oldpath = files.avatar.path
                console.log(oldpath)
                var newpath = form.uploadDir + fields.username + extname
                var issave = false
                if (!fs.existsSync(newpath)) {
                    fs.rename(oldpath, newpath, function(err) {
                        if (err) {
                            throw err
                        }
                    })
                    issave = true
                }
            }
            if (err) return reject(err)
            resolve({ fields: fields, files: files, newpath: newpath, oldpath: oldpath, extname: extname, issave: issave })
            })
        })
    }

    var body = await formidablePromise(ctx.req, null);

    var info = body.fields
    try {
        const user = await UserModel.getUserInfo(info.username);
        if (user != null) {
            ctx.body = {
                code: 409,
                msg: '该用户已存在',
                data: null 
            }
            if (fs.existsSync(body.oldpath)) {
                fs.unlinkSync(body.oldpath)
            }
            if (body.issave) {
                if (fs.existsSync(body.newpath)) {
                    fs.unlinkSync(body.newpath)
                }
            }
            return
        }
            
        const res = await UserModel.createUser(info, body.extname);
           
        ctx.status = 200;
        ctx.body = {
            code: 200,
            msg: 'success',
            data: res
        }
    } catch(err) {
        ctx.status = 500;
        ctx.body = {
            code: 500,
            msg: 'failed',
            data: err
        }
    }
}
```            

### 对已注册用户的头像的更新
最开始的操作时直接通过username，对“`....static/uploads/users/[username].extname`“的文件内容更新，但由于数据库中的用户头像的url是根据这个文件名来存储的，所以这就造成我们只是更改了图片内容，但在文件名和数据库中的数据并没有发生变化，这对后端可以接受，但前端通过查询用户头像，发现头像url没变，就误以为内容没有更新，所以页面的显示头像的地方并不会直接刷新更改图片内容，所以最后采用的方法是通过`username + 时间戳 + 图片扩展名`的方式来存储图片，然后再删除原来的头像图片，核心代码为：
```js
const user = await UserModel.getUserInfo(ctx.session.username)
var oldpath = user.avatar.replace(/http:\/\/139.196.79.193:3000\/uploads\/user\//,__dirname +  "\\static\\uploads\\user\\")
oldpath = oldpath.replace(/controller\\/, '')
if (fs.existsSync(oldpath)) {
    fs.unlinkSync(oldpath)
}
```
这将会删掉那张已存在的旧图片，然后再和注册时一样原理的存储新图片，并更新其文件名，就可以达到前后端同时更新头像的目的了。
