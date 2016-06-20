#Android多进程模式


##开启多进程模式
###方法有两种：  
* 在Manifest中指定android:process
* 非常规的方法，在jni中fork一个新的进程  

默认的进程名是包名  

###android:process
名称可以使用 ":xxx"：属于私有进程，其他应用的组件不可以和他跑在一个进程中  
也可以使用完整的包名：属于全局进程

###使用多进程会带来的问题
* 静态成员和单例模式完全失效
* 线程同步机制完全失效
* sharedPreferences可靠性下降
* Application 多次创建

