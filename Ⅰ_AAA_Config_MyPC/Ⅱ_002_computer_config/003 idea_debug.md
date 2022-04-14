## 1. 单线程
### 01 详细断点, 源断点
- shift + 左键    -->    **suspend 挂起 不勾选**
![[Pasted image 20220414211017.png]]
![[Pasted image 20220414210047.png]]
![[Pasted image 20220414210502.png]]

### 02 方法断点
![[Pasted image 20220414210821.png]]

- 如果打在接口上, 会自动进入实现类

### 03 异常断点
![[Pasted image 20220414211313.png]]


### 04 监控断点

## 2. 多线程
![[Pasted image 20220414212235.png]]
![[Pasted image 20220414212257.png]]

## 3. 进阶
### 01 条件表达式
![[Pasted image 20220414212811.png]]

![[Pasted image 20220414212820.png]]


### 02表达式解析_ evaluate(略)

### 03 避免操作资源_force return
![[Pasted image 20220414213619.png]]

### 04 其他功能
1. Show Execution Point - 回到当前断点处
2. Step Over - 下一行
3. Step Into - 进入方法(不包括源码)
4. Force Step Into - 强制进入一切方法
5. Step Out - 跳出
6. Run to Cursor - 定位到光标

### 05 stream debug
![[Pasted image 20220414220746.png]]


