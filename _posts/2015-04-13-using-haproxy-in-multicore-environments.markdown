---
layout: post
title:  "Using HAproxy in multi core environments"
date:  2015-04-13 19:09:00
categories: web
---

<a href="http://haproxy.1wt.eu/" rel="nofollow">HAproxy</a> is a great load balancing solution that we use at <a href="https://tr.instela.com">Instela</a>. We use HAProxy in a 8-Cores bare metal machine and we also use it to offload SSL encryption.

Although HAproxy is very effective solution in terms of CPU usage, SSL offloading obviously needs more CPU power, thus, using only one core could be easily a bottleneck. Apart from this, also we did not want to waste other cores and we decided to activate <i>not recommended</i> multi core support.

The configuration is pretty straightforward. At the `global` section of `haproxy.cfg`, we put these directives:

{% highlight python %}
    nbproc 15
        cpu-map 1 1
        cpu-map 2 2
        cpu-map 3 3
        cpu-map 4 4
        cpu-map 5 5
        cpu-map 6 6
        cpu-map 7 7
        cpu-map 8 8
        cpu-map 9 9
        cpu-map 10 10
        cpu-map 11 11
        cpu-map 12 12
        cpu-map 13 13
        cpu-map 14 14
        cpu-map 15 15
        stats bind-process 15
{% endhighlight %}

These directives tells HAProxy to use 15 threads (we have 16 virtual cores and we leave one of them free) and assigns each thread to a core.

The magic happens when we define frontends. `bind-process` directive at `frontend` section allows us to assign each frontend to a specific thread. For instance, we have a non-HTTPS frontend which is redirecting non-HTTPS traffic to HTTPS one and we assigned only one thread to this frontend:

{% highlight python %}
frontend instela
        mode http
{% endhighlight %}

On the other hand our heaviest frontend is dealing with the HTTPS traffic and SSL offloading, therefore we assigned the most of our threads to this frontend:

{% highlight python %}
frontend instela-ssl
        mode http
        option forwardfor except 127.0.0.1
        option  httplog
        option http-server-close
        bind-process 2 3 4 5 6 7 8 9 10 11
{% endhighlight %}

And our other threads are assigned to another frontends that has lighter traffic.

The configuration is so easy. But there are some important things to be considered when we use HAproxy in multithreaded mode. You must be aware that every thread has its own memory area and they are not aware of themselves. We can see this phrase in the HAproxy documentation more than once:

`Note : Consider not using this feature in multi-process mode (nbproc > 1)
         unless you know what you do : memory is not shared between the
         processes, which can result in random behaviours.`

The most common problem that we faced was that our backend servers were receiving multiple HTTP check requests. In the beginning we did not find out the reason of these unnecessary requests but after a while we realized that every thread was checking backend availability on its own because they were not be able to share the availability information between each other.

If you can ignore this random behaviour and you do not heavily need the statistics feature, you can use HAproxy in multithreading mode as we do.