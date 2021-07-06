#java后台启动jar包
# java后台启动jar包
1.当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出

```java
java -jar shareniu.jar
```
2.当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。

```java
java -jar shareniu.jar &
```

&amp;代表在后台运行

3.不挂断运行命令,当账户退出或终端关闭时,程序仍然运行，当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件

```java
nohup java -jar shareniu.jar &
```
4.将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中

```java
nohup java -jar shareniu.jar >/dev/null  & 
```

/dev/null 表示空设备文件，一般使用.out后缀，或.file后缀

5.将错误重定向到标准输出上

```java
nohup java -jar shareniu.jar >/dev/null 2>&1 &
```
6.指定启动端口

```java
java -jar xxx.jar --server.port=80
```
7.指定堆内存

```java
nohup java -Xms2000m -Xmx3000m -jar apilog-0.0.1-SNAPSHOT.jar 
```

>  **》》》博主长期更新学习心得，推荐点赞关注！！！** **》》》若有错误之处，请在评论区留言，谢谢！！！** 
