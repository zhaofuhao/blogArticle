## cmd窗口实现命令启动idea

用过vscode的朋友知道在cmd命令窗口下 输入code .就可以打开vscode 并且打开 你cmd命令下的当前文件夹，今天来分享一下如何使用idea实现这种操作

1. 第一步安装idea的时候勾选两个选项 将idea加入到系统环境变量中

![image-20230826160240508](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230826160240508.png)

2. 勾选好以后就可以实现了输入`idea64.exe .` 就可以在命令行让idea打开指定文件夹了

```cmd
F:\代码文件\开源项目\jeecgboot\后端\新版本\jeecg-boot>idea64.exe . 
```

> 注意：如果使用的是360文件夹作为系统文件夹管理工具的话 还得需要多做一步操作才能使用这个命令 需要去360文件夹快捷方式勾选以管理员方式运行。

![image-20230826160632720](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230826160632720.png)

![image-20230826160653978](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230826160653978.png)

![image-20230826160722670](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230826160722670.png)