---
title: python-pyqt多线程
date: 2019-02-03 10:41:34
tags: [python]
categories: python

---
这篇讲一下项目中用到的QThread多线程，主要是维护了一个pyqt界面，并使用多线程做人脸检测、识别。
同时也是在一个人脸识别的系统项目中学习到的对pyqt界面以及pyqt多线程的编程应用，代码开源：[https://github.com/k-miracle/ARC_Face](https://github.com/k-miracle/ARC_Face)

# 1 QThread
继承线程类，覆写`run()`
```
class Thread(QThread):
   def __init__(self):
      super().__init__()
   def run(self):
      # 线程相关代码		
      pass
# 创建一个新的线程
thread = Thread()
thread.start() # 启动线程
```
start()启动线程，调用run()，当run()退出后线程结束
## 1.1 管理线程
当线程`start()`或`finish()`时，`QThread`会通过信号通知；或者可以使用`isFinished()`和`isRunning()`来查询线程状态。
使用`exit()`或`quit()`来停止线程
使用`wait()`来阻塞线程，直到另一个线程完成【一般监听`finish`信号，而不是使用`wait()`】
使用`sleep()`,`msleep()`,`usleep()`进行秒，毫秒，微秒的睡眠【一般使用`QTimer`，而不是使用`sleep()`】
# 2 信号与槽
pyqt中当事件发生改变时，就会发处信号。信号会触发所有与该事件（信号）相关的函数（槽）。一个信号可以连接多个槽，一个槽可以监听多个信号。
**注意**：QThread类中常用的信号有`start()`,`finished()`，分别表示在run函数运行之前和运行完run之后线程发射的信号。
## 2.1 自定义信号与槽
- 定义信号
- 定义槽
- 连接信号与槽
- 发射信号
### 2.1.1 定义信号
通过`类成员`变量定义信号对象
```
class MyWidget(QWidget):  
    # 无参数的信号
    Signal_NoParameters = pyqtSignal()     
    # 带一个参数(整数)的信号      
    Signal_OneParameter = pyqtSignal(int)         
    # 带一个参数(整数或者字符串)的重载版本的信号        
    Signal_OneParameter_Overload = pyqtSignal([int],[str])
    # 带两个参数(整数,字符串)的信号      
    Signal_TwoParameters = pyqtSignal(int,str)    
    # 带两个参数(整数,整数)的重载版本的信号      
    Signal_TwoParameters_Overload = pyqtSignal(int,int) 
```
### 2.1.2 定义槽函数
定义槽函数，有多个不同的输入参数
```
class MyWidget(QWidget):  
    def setValue_NoParameters(self):   
    	'''无参数的槽函数'''  
    	pass  
    
    def setValue_OneParameter(self,nIndex):   
    	'''带一个参数(整数)的槽函数'''  
    	pass

    def setValue_OneParameter_String(self,szIndex):   
    	'''带一个参数(字符串)的槽函数'''  
	pass 
    
    def setValue_TwoParameters(self,x,y):   
    	'''带两个参数(整数,整数)的槽函数'''  
	pass  
    
    def setValue_TwoParameters_String(self,x,szY):
    	'''带两个参数(整数,字符串)槽函数''' 
	pass
```
### 2.1.3 连接信号与槽
通过`connect`连接:`信号.connect(槽)`
```
app = QApplication(sys.argv)   
widget = MyWidget()   

# 连接无参数的信号
widget.Signal_NoParameters.connect(self.setValue_NoParameters )                                          
# 连接带一个整数参数的信号
widget.Signal_OneParameter.connect(self.setValue_OneParameter)                                         
# 连接带一个整数参数，经过重载的信号
widget.Signal_OneParameter_Overload[int].connect(self.setValue_OneParameter)                              
# 连接带一个整数参数，经过重载的信号
widget.Signal_OneParameter_Overload[str].connect(self.setValue_OneParameter_String )

# 连接一个信号，它有两个整数参数
widget.Signal_TwoParameters.connect(self.setValue_TwoParameters )                                        
# 连接带两个参数(整数,整数)的重载版本的信号
widget.Signal_TwoParameters_Overload[int,int].connect(self.setValue_TwoParameters )                      

# 连接带两个参数(整数,字符串)的重载版本的信号
widget.Signal_TwoParameters_Overload[int,str].connect(self.setValue_TwoParameters_String )              

widget.show() 
```
### 2.1.4 发射信号
在需要发出信号的代码段，执行`信号.emit()`，即发处信号，`connect`连接到槽，接收该信号
```
class MyWidget(QWidget):  
    def mousePressEvent(self, event):  
    	# 发射无参数的信号
    	self.Signal_NoParameters.emit() 
        # 发射带一个参数(整数)的信号
	self.Signal_OneParameter.emit(1) 
	# 发射带一个参数(整数)的重载版本的信号
	self.Signal_OneParameter_Overload.emit(1)
	# 发射带一个参数(字符串)的重载版本的信号
	self.Signal_OneParameter_Overload.emit("abc")
	# 发射带两个参数(整数,字符串)的信号
	self.Signal_TwoParameters.emit(1,"abc")
	# 发射带两个参数(整数,整数)的重载版本的信号
	self.Signal_TwoParameters_Overload.emit(1,2)
	# 发射带两个参数(整数,字符串)的重载版本的信号
	self.Signal_TwoParameters_Overload.emit (1,"abc")
```
