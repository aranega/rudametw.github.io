---
layout: blog-post
title: NSLU2 Linux Gateway
place: Rennes, France
categories: [linux, nslu2, work-in-progress]
---

<div class="container">
    <div class="row">
        <div class="col-md-4">
			<img src="/img/wip3.jpg" alt="Work in progress"/>
		</div>
        <div class="col-md-7">
			<img src="/img/nslu2.jpg" alt="NSLU2.jpg"/>
        </div>
    </div>
</div>

---

It's easy to setup additional IP addresses on Debian Linux. This is particularly useful for the NSLU which doesn't have a display so you need to remotely connect to it.

Having more than one virtual network interface allows us to have both a DHCP address and a static IP address, making the NSLU2 accessible pretty much always.

<!--more-->

###On the nslu
I'm supposing you have somekind of IP address already setup. Also, you should have a working DNS, although this shouldn't impact any of the following.

Once you can ping the gateway machine, the gateway must be added so that the nslu knows that he can attain addresses that are beyond his own network.

Command is 
{% highlight bash%}
	route add default gw 192.168.0.100
{% endhighlight %}

Then the `/etc/resolv.conf` must be edited to the same nameserver as the other box. Note that nslu should be able to ping this nameserver once the gateway is setup.

For setting up the gateway we must enable ip forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

###Additional virtual network interface

If we want to setup alias cards, we can do something along the lines of
```bash
ifconfig eth0:N address    netmask
```

However, it's probably easier to just edit the `/etc/network/interfaces` file by adding something like this

{% highlight bash%}
iface eth0:0 inet static
		address 192.168.0.100
		netmask 255.255.255.0
		broadcast 192.168.0.255
		network 192.168.0.0
		pre-up echo "*** Starting eth0:0 alias ***"
		post-up echo "*** Alias eth0:0 started ***"
		#post-up route add -net 192.168.0.0 dev eth0
		#post-up route add -host 192.168.0.10 dev eth0:1
{% endhighlight %}

This allows us to have a static IP address in addition to whatever other configured addresses we might have. I used it to have both a DHCP assigned address and a static local network address, allowing me to always get access to the NSLU2, even if it's not getting an address through DHCP.

Finally, must set up iptables to the correct rules!!!

I found and change a script to setup the tables to allow 

