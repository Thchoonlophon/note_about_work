# 守护进程概念
---
## 需求背景
之前公司开会讨论关于项目的异常处理日志，以及抛出异常之后及时发送邮件进行通知。
在讨论过程中有同事提出如果程序执行出错抛出异常可以发送邮件通知，那万一程序在运行中Python崩溃了该怎么处理，Python崩溃的话我们的异常抛出机制再怎么完善邮件也发不出去的，然后莫名其妙这个烂摊子就交给我负责了。   

---
## 需求处理
用Python来监测Python有没有崩溃显然是个很扯淡的一个方案，由于公司项目部署在的服务器的系统是Windows(不知道为啥不用Linux)真是令人费解，那也得做啊，写个.bat脚本吧。
![img](https://wx4.sinaimg.cn/mw690/005RsMBrly1furllf37clj3096096q2z.jpg)

```bat
@echo off 

rem 定义需监控程序的进程名和程序路径，可根据需要进行修改
set AppName=python.exe
set AppPath="E:\theVirtualEnvs\Django_test\Scripts\"

rem 已实际状况来确定需不需要这一步
rem 用于激活virtualenv创建的虚拟环境
set source="E:\theVirtualEnvs\Django_test\Scripts\activate.bat"

rem 用于测试使用的项目中的manage.py文件
set Document="E:\Downloads\Soical_site\manage.py"
rem 用于发送邮件，路径需按照实际需求调整。
set send_mail="E:\send_log\collapse.py"
title Process monitoring

cls

echo.

echo Start of process monitoring

echo.

rem 定义循环体

:startjc

   rem 从进程列表中查找指定进程

   rem  下面语句也可写成 qprocess %AppName% >nul
   qprocess|findstr /i %AppName% >nul

   rem 变量errorlevel的值等于0表示查找到进程，否则没有查找到进程
   if %errorlevel%==0 (

         echo ^>%date:~0,10% %time:~0,8% The program is running

    )else (

           echo ^>%date:~0,10% %time:~0,8% No process was found

           echo ^>%date:~0,10% %time:~0,8% Restarting the program

           rem 用于测试，实际投入使用需要项目部署环境的具体情况
           rem 激活虚拟环境，调用虚拟环境内Python 运行manage.py runserver
           start %source% %AppPath%%AppName% %Document% runserver >nul && echo ^>%date:~0,10% %time:~0,8% Successful startup
           rem 发送邮件
           start %AppPath%%AppName% %send_mail% >nul

   )

   rem 用ping命令来实现延时运行

   for /l %%i in (1,1,10) do ping -n 1 -w 1000 168.20.0.1>nul

   goto startjc

echo on
```

