## 可能遇到的问题
1. **账号密码错误:**
	控制面板\\所有控制面板项\\凭据管理器 -> 普通凭据
	Internet地址或网络地址:  git:https://gitee.com
	用户名: gitee 账号
	密码: gitee 密码
	
## 01 tortoise步骤
1. 先创建版本库
2. 提交分支, 注意: 提交信息要写
3. 右键 Git Bash Here -> 输入:(gitee仓库里有教程)
	 git remote add origin https://gitee.com/saddyfire/obsidian_java.git
4. git push -u origin master


## 02 idea删除git项目
打开菜单：Settings --> Version Control

选中项目，删除，ok

这时并没有真正删除项目的git信息，还需要到项目的根目录中，删除两个目录：.git 和 .idea

这是git信息才算真正的删除完毕。。。