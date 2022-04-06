nginx设置

 ```
server {
	listen 80;
	server_name _;
	location / {
		..................
		proxy_pass http://127.0.0.1:8000/;
		# $host 变量，Host 为变量名
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	
	}
}
```
 

在nginx中添加配置：
```
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-For $http_x_forwarded_for;
```


解释：
X-Real-IP $remote_addr ：

将用户的真实IP存放到X-Real-IP这个变量中（变量名可以自定义）

获取：
request.getHeader("X-Real-IP")

X-Forwarded-For $proxy_add_x_forwarded_for：

添加用户的真实IP存放到X-Forwarded-For变量中（变量名可以自定义），如果搭建了两台nginx在不同的ip上，将会获取到用户的真实IP和第一台nginx的ip,以“，”隔开。

`$proxy_add_x_forwarded_for`中包含了客户端请求头中的"X-Forwarded-For"和$remote_addr两个部分。

如果搭建了两台nginx在不同的ip上：

经过第一个nginx的时候，此时"X-Forwarded-For"为null，"X-Forwarded-For"默认为null，只有$remote_addr，$remote_addr中就是用户的真实ip，所以此时的"X-Forwarded-For"中的值就是用户的真实ip，
经过第二个nginx的时候，此时"X-Forwarded-For"中包含了用户的真实ip，$remote_addr值为上一个nginx的ip
所以经过两个nginx之后，获取到的值是：客户端ip和上一个nginx的ip

获取：
request.getHeader("X-Forwarded-For")

X-Forwarded-For $http_x_forwarded_for：

将用户的真实IP存放到X-Forwarded-For这个变量中（变量名可以自定义）

获取：
request.getHeader("X-Forwarded-For")