jenkins发版流程
# 流程
## 1. 登陆
https://jenkins.wozaihcm.com/
账户：sqian
密码：询问@王斌

## 2. stage-maintenance（prod-maintenance）

status：on
timeStr: 恢复服务的时间

验证生效：https://app.wozaihcm.com/#/home?_k=laqbvq
（http://app.wozai.hr/#/home?_k=pdd0zu）

## 3. stage-front(prod-front)
commit: commitId
(machineName:prod-nginx-01-1a ,prod-nginx-01-1b 二选一)

 **注意点:**
 - zk有没有写入成功
 观察日志中以下内容
 ```
 set /fe/css/onboarding/version 3055298BF3507787614F394F5954FFEE

 get /fe/css/onboarding/version
 3055298BF3507787614F394F5954FFEE
 ```


 - zk节点共有
 ```
 /fe/css/onboarding/version 入职邀请链接点入
 /fe/css/app/version 首页
 /fe/css/auth/version 登陆页
 /fe/css/admin/version 管理员页（使用 huanchen@sqian.com账户 密码：Sq1234567）
 /fe/js/onboarding/version
 /fe/js/app/version
 /fe/js/auth/version
 /fe/js/admin/version
```

 - ansible命令有无执行成功
**"changed":"true"**
```
 TASK: [Echo] ******************************************************************
 changed: [nginx-01-1b] => {"changed": true, "cmd": "whoami ", "delta": "0:00:00.001835", "end": "2016-11-01 00:16:02.647441", "rc": 0, "start": "2016-11-01 00:16:02.645606", "stderr": "", "stdout": "ubuntu"}
 TASK: [Copy_Package] **********************************************************
 changed: [nginx-01-1b] => {"changed": true, "checksum": "c3404672e0cb74f0e752c00147dcbeb9ddd72dae", "dest": "/www/hr/fe.tar.gz", "gid": 1000, "group": "ubuntu", "md5sum": "56c1ac5cf13ffa50cb9fac23340b8824", "mode": "0644", "owner": "ubuntu", "size": 33300480, "src": "/home/ubuntu/.ansible/tmp/ansible-tmp-1477930521.88-274066903880559/source", "state": "file", "uid": 1000}
 TASK: [Unpack] ****************************************************************
 changed: [nginx-01-1b] => {"changed": true, "cmd": "cd '/www/hr' && rm -rf static *.html && tar -xf '/www/hr/fe.tar.gz' -C '/www/hr' && rm fe.tar.gz ", "delta": "0:00:00.154408", "end": "2016-11-01 00:16:04.292450", "rc": 0, "start": "2016-11-01 00:16:04.138042", "stderr": "", "stdout": ""}
```

 - zk有无写入成功可以在产品中验证

# 4. stage-wozai(prod-wozai)

  1. 0.9.x checkList( sql、需要deploy的包、prod环境需要改的配置文件)
  2. commit:commitId
     serviceName:api feed review third-party user
     (machineName: prod-app-1a-01,prod-app-1b-01)

  3. 发版顺序
	最先发third-party 最后发api

  - ansible命令有无执行成功（同前端）

  成功标志：
  ```
  =============================================\ntimeStr:2016-11-03 16:50:33 its timeStamp :1478163033 systemTimeStamp :1478163016 log:2016-11-03 16:50:33 INFO h.w.s.u.s.UserServerApplication - Started UserServerApplication in 16.417 seconds (JVM running for 17.116)\n=============================================\ndeltaTime:17s\nreturnValue of checkLogContent()
  ```



## 附
### 1. 新建stage分支流程
```create branch dev->stage-0.9.xx```

### 2. 新建prod分支流程
```
stage-0.9.xx pom更新一下
1.修改stage-0.9.xx-wangbin分支上 third-party-client 版本0.9.9-SNAPSHOT -> 0.9.9
2.修改所有依赖上述client的地方的版本号
3.提pr stage-0.9.xx-wangbin -> stage-0.9.xx
4.再发一次stage
5.create pull request: stage-0.9.xx -> prod
发版完成后：
6.create pull request: stage-0.9.xx -> dev
```
