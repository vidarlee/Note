- SSH连接不通（SUSE）
-
  `#service sshd status  / systemctl status sshd`

  如果service开启了，就确认防火墙

  `#iptables -nL | grep 22`

  如果没有任何输出说明22端口被拦截。

  `#yast firewall`

  Allowed Services -> Service to Allow  -> Add Secure Shell Server|Open ports for Secure Shell Server

- 挂载安装包
  > command-not-found - A command-not-found handler
  ```
  #cnf git
  #zypper install git-core
  #cnf sar
  #zypper install sysstat
  #zypper install gcc
  ```

- sudo no password
  ```
  # visudo
  user ALL=(root) NOPASSWD: ALL  #add this line
  ```
