### General Information ###

Advanced Reporter for memcached is a memcache server traffic analyzer for Linux which provides information about the most active clients and object keys in your memcached server.  It can provide stats like:

  * Hot keys - e.g. which keys are used the most often in certain memcached servers.
  * Hot clients - e.g. which clients (based on their IP and port) accesses specific memcached servers most often.
  * Hot clients on a per-key basis - e.g. which clients has accessed a specific key the most.

The code has two parts, one is a kernel patch to intercept traffic from TCP/IP stack, another part is a user space daemon that does analysis on the data collected from kernel part.Since the tool includes a TCP/UDP stream-level interceptor and a high- speed collector inside kernel, it would require you to patch and re-build your Linux kernel before you can use it.

### Memory Requirement ###

Advanced Reporter was initially developed for Gear6's memcache appliance, it has the assumption of large memory server environment. By large memory, we mean 10G or more. It requires at least 300M memory to load, and for each target server added, it requires an addition 300M to collect and hold data for analysis.

### Kernel Supported ###

The kernel patch was developed for Linux 2.6.18.8. Basically we put hooks inside TCP/IP stack to collect memcache related traffic to target servers, by marking related opening socks for the servers, and collect data from socket level whenever the servers read.

With minor twist, this approach should work for kernels up to 2.6.24.7. Kernel 2.6.25 starts to add a network namespace field in 'struct sock'. If you won't be using the network namespace feature, the code still holds for kernel up to 2.6.18.9. Otherwise you would need to add network namespace (struct net) related processing to make the code namespace knowledgeable.

Linux Kernels starting from 2.6.29 (released in March, 2009) have major changes to the UDP and TCP hash tables that hold opening socks, thus the interceptor part of the code needs major revision to work under these kernels.

### Architecture ###

The reporter is comprised of code in both user space and kernel space. For details please see the enclosed block diagram:

![http://www.mirreo.com/adv_reporter_1.png](http://www.mirreo.com/adv_reporter_1.png)

The user space code doesn't change across Kernel versions, but it depends on the kernel space code to compile. The kernel space code changes for different kernel versions.

### Kernel Space ###

The kernel part of the code can be divided into largely two parts, the Collector and the Interceptor.

The collector does data processing for target servers, the interceptor gets traffic from TCP/IP stack, does simply parsing, then pass the data to the collector. These two parts are held together under the mcr device file.

What really changed for different kernel versions are the interceptor files, namely those hooks into the TCP/IP stack. These include:
```
   ./include/linux/sock.h
   ./net/core/sock.c
   ./net/ipv4/udp.c
   ./net/ipv4/inet_connection_sock.c
   ./net/socket.c
```
Other files that may need to be changed to support network namespace include:
```
   ./drivers/mcr/mcr_interceptor.c
   ./drivers/mcr/mcr_intercept_tcp.c
   ./drivers/mcr/mcr_intercept_udp.c 
   ./drivers/mcr/mcr_target_manager.c
   ./drivers/mcr/mcr_parser.c
   and corresponding header files. 
```