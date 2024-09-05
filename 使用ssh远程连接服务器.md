# 使用ssh远程连接服务器

---



## 1.切换成root用户
sudo …
## 2.启动ssh服务

service ssh star

## 3.检查SSH服务是否启动成功
ps -e| grep ssh 
(出现sshd说明启动成功) 

## 4.启用SSH命令

systemctl enable