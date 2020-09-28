# AspNetCore程序部署至CentOS7中

## 运行环境
* CentOS 7
* nginx 1.18.0
* mariadb 5.5.65
* dotnet SDK 3.1.402

## 主要步骤
1. 关闭SELINUX
2. 设置防火墙
3. 安装nginx
4. 安装mariadb
5. 安装dotnet runtime
6. 发布程序

---
### **关闭SELINUX**
SELINUX是Linux系统的安全体系结构，它允许管理员对访问系统的人有更多的控制。它最初由美国国家安全局(NSA)开发，是使用Linux安全模块(LSM)对Linux内核进行的一系列补丁。

为什么要关闭：因为设置起来很麻烦，所以一般默认关闭。

查看SELINUX的状态：  
` getenforce `

临时关闭：  
` sudo setenforce 0 `

永久关闭：  
` sudo vim /etc/selinux/config `        
将SELINUX参数设置为disabled  
保存修改后重启系统  
` sudo shutdown -r now `

确认SELINUX已关闭：  
` getenforce `


### **设置防火墙**
查看防火墙状态：  
`systemctl status firewalld`

如果显示未启用则输入：  
`sudo systemctl enable firewalld # 设置开机自启动  `  
`sudo systemctl start firewalld # 立即启动`

查看已开放的端口：  
`sudo firewall-cmd --list-port --zone=public`

开放目的端口：  
`sudo firewall-cmd --add-port=[target port]\tcp --zone=public --permanent # --permanent表示永久开放`

重启防火墙：  
`sudo firewall-cmd --reload`

确认端口已开放：  
` sudo firewall-cmd --list-port --zone=public `

### **安装nginx**
安装过程参照[官方文档](https://nginx.org/en/linux_packages.html#RHEL-CentOS)

开启服务：  
` sudo systemctl enable nginx `  
` sudo systemctl start nginx `

开放80端口：  
`sudo firewall-cmd --add-port=80/tcp --zone=public --permanent # --permanent表示永久开放`  
`sudo firewall-cmd --reload`

查看本机IP地址（有多个IP，哪个是？）：  
` ip addr `

在浏览器中输入上面显示的IP地址，如果出现 **Welcome to nginx!** 表示nginx已启动。

日志位置：  
/var/log/nginx/error.log  
/var/log/nginx/access.log


### **安装mariadb**
[官方文档](https://mariadb.com/resources/blog/installing-mariadb-10-on-centos-7-rhel-7/)  
使用yum安装：  
` sudo yum install MariaDB-server `  
启用服务：  
` sudo systemctl enable mariadb `  
` sudo systemctl start mariadb `

配置mariadb：  
` sudo mysql_secure_installation `  
需要设置root用户密码，其余选项根据实际需要选择（一般选择y即可）

登录数据库：  
` mysql -u root -p `  
然后输入密码进入mysql命令行环境

创建一般用户：  
` create user '[user]'; ` 

设置用户密码：  
` set password for '[user]' = PASSWORD('[password]'); ` 

刷新权限：  
` FLUSH PRIVILEGES; `

查看已创建的用户：  
` use mysql; `  
` select host, user from user; `  

新建database：  
` create database [name]; `

开放database权限给某用户：  
` grant all privileges on [database].* to '[user]'; `

### **安装dotnet core**
具体过程参照[官方文档](https://docs.microsoft.com/en-us/dotnet/core/install/linux-centos#centos-7-)

注意事项：  
1. 配置文件中的监听端口要开放；
2. 服务文件中的WorkingDirectory，ExecStart，User要根据实际情况进行设置；
3. 服务文件要放在/usr/lib/systemd/system/中才能启用；
4. nginx配置文件的监听端口不能和程序的监听端口一样。

### **发布程序**
1. 修改appsettings.json中的ConnectionStrings：
```json
"ConnectionStrings": {
    "Context": "Server=localhost;Port=3306;Database=myDB;User=[user name];Password=[password];"
  }
```  
2. 在VS2019中将程序发布到某个文件夹中；
3. 再将该文件夹中的内容通过WinSCP传输到CentOS的某个文件夹中（要与上一步骤中设置的 WorkingDirectory 一致）；
4. 最后在浏览器中输入[IP:Port]即可访问程序。