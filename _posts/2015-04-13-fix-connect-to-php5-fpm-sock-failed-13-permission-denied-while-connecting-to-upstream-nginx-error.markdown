---
layout: post
title:  "How to fix connect() to php5-fpm.sock failed (13: Permission denied) while connecting to upstream Nginx error"
date:  2015-04-13 17:52:00
categories: web
---
This is a very common problem after upgrading PHP especially on Ubuntu systems. I am going to explain how to fix this basic but hard to understand problem.

<h3>Some background</h3>

<a href="https://en.wikipedia.org/wiki/Unix_domain_socket">Unix Sockets</a> provide us an alternative method to TCP/IP in the FastCGI implementations. Using Unix sockets removes all the limitations of TCP/IP and as they know that they are executing on the same system, they can avoid some overheads such as checks and operations (like routing) and this makes them faster and lighter than IP sockets. So if you are executing two processes on same host this is a better option than IP sockets. Unix sockets are like regular files and therefore all the file permissions are applied on them.

<h3>Solution</h3>

In this specific problem, you would see these lines in your nginx log.

`2015/12/04 11:22:45 [crit] 4202#0: *1 connect() to unix:/var/run/php5-fpm.sock failed (13: Permission denied) while connecting to upstream, client: xx.xxx.xx.xx, server: localhost, request: "GET / HTTP/1.1", upstream: "fastcgi://unix:/var/run/php5-fpm.sock:", host: "xx.xx.xx.xx"`

It basically means that nginx process cannot access the file located on /var/run/php5-fpm.sock.

<a href="http://stackoverflow.com/questions/23443398/nginx-error-connect-to-php5-fpm-sock-failed-13-permission-denied/27879922" target="_blank" rel="nofollow">Some folks</a> suggest removing permission checks for the sockets, it is obvious that this method creates some security problems.

So, the right solution would be permitting nginx's running user to socket file by this command. So, first check which user runs nginx. As of Ubuntu 12.04 nginx runs by `nginx` user which is not a member of `www-data` group, but the socket file can be accessed by this group. So, let's simply add the user to this group:

`usermod -a -G www-data nginx`

then restarting nginx and php5-fpm daemons solves the problem completely.