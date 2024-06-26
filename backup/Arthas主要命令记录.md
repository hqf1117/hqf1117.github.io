# Arthas

java在线诊断工具

## 运行环境要求

JDK6+，支持linux/Mac/windows

## 安装启动

```shell
curl -O https://arthas.aliyun.com/arthas-boot.jar
#端口可能被占用的情况下
java -jar arthas-boot.jar --telnet-port 9998 --http-port -1
```

## 命令

### 常用命令

- dashborad:查看基础信息面板，定时刷新。按q退出。

<img width="1470" alt="image-20240527144437192" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/b1439d20-3168-4638-ae71-d05b2a0a45a2">


- thread：
  - 所有线程的状态。
  - thread [pid]：指定pid的线程状态
  - -b:阻塞中的线程
  - -nx：最忙的x个线程
  - -i：指定取样间隔，单位ms
  - --state [value] : 查看指定状态的线程
- jad：反编译某个文件，可显示加载类的类加载器，以及所属jar包名称和类代码。
- watch：watch 包名 方法名 returnObj（OGNL表达式）

### 基础命令

- cls：清屏
- pwd
- cat:根据pwd显示的当前路径，来判断cat文件的路径。可以不q的情况下查看log日志等。
- grep：
  - -n：显示行号
  - -i：忽略大小写
  - -m x：最大显示x行
  - -e "reg"：正则表达式查找

- session：显示sessionId
- reset：还原所有增强过的类/指定的类/通配符指定的类，退出arthas时会自动重置
- version：查看arthas版本号
- history：历史
- quit：退出当前session
- stop：退出arthas服务，关闭其他所有session
- keymap：快捷键查看

### jvm命令

- jvm：查看jvm相关的状态信息

- sysprop：查看系统参数

- sysenv：env参数

  - sysenv [name]:查看指定参数

- vmoption： 查看所有jvm选项

  - vmoption [name]:查看指定名称的参数
  - vmoption [name] [value]:修改指定的vm选项值

- getstatic：实时获取静态属性

- vmtool： 利用`JVMTI`接口，实现查询内存对象，强制 GC 等功能。since 3.5.1

  ```shell
  #查询内存对象
  $ vmtool --action getInstances --className java.lang.String --limit 10
  @String[][
      @String[com/taobao/arthas/core/shell/session/Session],
      @String[com.taobao.arthas.core.shell.session.Session],
      @String[com/taobao/arthas/core/shell/session/Session],
      @String[com/taobao/arthas/core/shell/session/Session],
      @String[com/taobao/arthas/core/shell/session/Session.class],
      @String[com/taobao/arthas/core/shell/session/Session.class],
      @String[com/taobao/arthas/core/shell/session/Session.class],
      @String[com/],
      @String[java/util/concurrent/ConcurrentHashMap$ValueIterator],
      @String[java/util/concurrent/locks/LockSupport],
  ]
  #强制gc
  $ vmtool --action forceGc
  #强制打断线程
  $ vmtool --action interruptThread -t 1
  ```

  

- ==**ognl：**==遵循ognl语法