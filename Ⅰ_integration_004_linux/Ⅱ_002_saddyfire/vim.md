#### 01 基础指令
#### 普通模式
`i` -> insert    (在光标前面插入)   ->   `shift + i`
`a` -> apend   (在光标后面插入)   ->   `shift + a`
`o` -> over  (在光标下一行插入)   ->    `shift + o` (在当前行上一行插入)

`hl` (左右)  `jk` (下上)

`w` -> word    (跳到下一个单词的开头)
`b` -> behind    (跳到上一个单词的开头)
`gg`   (跳到文件最上方)
`G`   (跳到文件最下方)

`ctrl + u`   (向上翻页)
`ctrl + d`   (向下翻页)
`fr`   (移动到最近的r的位置)

`y` -> yank(拉)   (复制)
例如:
`yaw` -> yank all words  (复制整个单词)
`y4j`   (复制了下4行)
`p` -> paste   (粘贴)

`d` -> delete   (删除)
例如:
`dj`   (删除当前行和下一行)

`u` -> undo   (撤销)
`ctrl + r` -> 取消撤回

`c` -> change (改变)
例如: caw -> change all word   (改变整个单词)  (删除当前这个词并进入输入模式)
`cc`   (删除这一行)
`c4j`   (删除下4行)
#### 输入模式


#### 命令模式
:wq -> write quite

#### 可使模式: 主要是用于选中一段内容
v -> visual   (进入可视模式)





###### vim 改键位

cd
mkdir .vim
cd .vim
vim vimrc

```vim
noremap n h  # 查找下一个位置
noremap u k
noremap e j
noremap i l

noremap k i
noremap K I
noremap l u

map S :w<CR>    # 保存
map s <nop>     # 小写s原先是删除当前字符并进入插入模式
map Q :q<CR>
map R 
```


