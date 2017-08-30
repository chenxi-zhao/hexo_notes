### 什么是aop
![](http://static.tmaczhao.cn/images/663b9126d71fd8e315591c43a7812b40.jpg)


### aop框架种类
- AspectJ
- JBoss AOP
- Spring AOP
使用aop可以做的事情有很多。
- 性能监控，在方法调用前后记录调用时间，方法执行太长或超时报警。
- 缓存代理，缓存某方法的返回值，下次执行该方法时，直接从缓存里获取。
- 软件破解，使用AOP修改软件的验证类的判断逻辑。
- 记录日志，在方法执行前后记录系统日志。
- 工作流系统，工作流系统需要将业务代码和流程引擎代码混合在一起执行，那么我们可以使用AOP将其分离，并动态挂接业务。
- 权限验证，方法执行前验证是否有权限执行当前方法，没有则抛出没有权限执行异常，由业务代码捕捉。

观察一下传统编码方式与使用aop的区别:
![](http://static.tmaczhao.cn/images/9d9e9546017af4ca2eefdde4151919d4.jpg)


### aop核心概念
![](http://static.tmaczhao.cn/images/aee8c482a492ec0dd44f99ad4b67c849.jpg)


### 拦截器和过滤器
![](http://static.tmaczhao.cn/images/b005523c8ee49dd4e88609b0225eae5b.jpg)



