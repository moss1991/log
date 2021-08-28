## 第一：备份源文件

```
cd /etc/apt/
```

然后会显示下面的源文件sources.list

输入命令行：(cp为copy的意思，就是将source.list备份到source.list.bak）

```
sudo  cp source.list source.list.bak
```

## 第二：替换清华源：

```
https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
```

打开链接：（ubuntu对应的清华源，有教程，注意选对自己的ubuntu型号）

打开文件：(gedit是用于更改系统文件内容的一个命令）

```
sudo gedit sources.list     打开文件后，将链接所说的清华源内容粘贴到source.list里面，然后保存。
```

## 第三：更新清华源
```
sudo apt-get update  更新清华源
sudo apt-get  upgrade  更新软件
```
