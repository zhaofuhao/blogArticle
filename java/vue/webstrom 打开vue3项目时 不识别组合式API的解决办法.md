## webstrom 打开vue3项目时 不识别组合式API的解决办法

> 最近在学vue3的时候 用vuecll脚手架创建vue3项目后 用webstrom打开后会提示一些错误 代码可以正常运行

![image-20230216205716759](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230216205716759.png)

>import 导入vue的组合式api 会提示无法解析 

**解决办法**

右键`node_modules`文件夹选择`Mark Directory as`，最后选择`Not Excluded` 等待刷新就可以了 

![image-20230216210744102](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230216210744102.png)

如果还不行 删除.idea文件夹 重启就可以了

