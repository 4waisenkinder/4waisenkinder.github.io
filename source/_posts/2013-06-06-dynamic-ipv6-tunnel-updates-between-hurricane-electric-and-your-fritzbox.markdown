---
layout: post
title: "Dynamic IPV6 Tunnel Updates Between Hurricane Electric And Your Fritzbox"
date: 2013-06-06 11:03
comments: true
categories:
- ipv6
---
TL;DR:
------
Associate your tunnel with a dynamically updated hostname from a subomain at
dns.he.net, use that host for fritzbox'es user-defined dynamic dns feature and
hardcode your tunnelid into the "Dynamic DNS provider" url.

The Problem:
============
<!-- more -->

[Hurricane Electric (he.net)](https://he.net) provides excellent free IPv6
tunnels at [tunnelbroker.net](https://tunnelbroker.net) together with free dns
hosting.  If you don't have an account yet, head over there now and open one up.
Do it now, you won't regret it. I'll wait.

Users of the popular [FRITZ!Box](http://www.avm.de/de/Produkte/FRITZBox/) home
router enjoy some ipv6 support out of the box, i.e.
[SixXs.net](http://www.sixxs.net)-tunnels can be configured via the gui:

{% img center /images/blog/bkw/fritzbox-sixx.png 665 177 'fritzbox sixxs tunnel config gui' 'fritzbox gui screenshot' %}

SixXs.net supports a protocol to remotely manage your tunnel endpoint.
However, he.net tunnels use the plain [6in4](http://en.wikipedia.org/wiki/6in4)
mechanism, which is designed for static endpoints. 

{% img center /images/blog/bkw/fritzbox-6in4.png 553 177 'fritzbox 6in4 tunnel config gui' 'fritzbox gui screenshot' %}

This implies your remote tunnel endpoint is configured to point to one specific
ip address at your end, presumably the one you had when you created the tunnel.

__This means that whenever your ISP gives you a new ip, your tunnel stops working.__


To remedy this, he.net provides a
[dyndns compatible update mechanism](http://www.tunnelbroker.net/forums/index.php?topic=1994.0)
to update your tunnel endpoint remotely. Fritzbox also supports adding
user-defined dyndns providers. So the naïve approach is to add he.net's url
into the gui and use the tunnel id as a hostname, like
[the documentation](http://www.tunnelbroker.net/forums/index.php?topic=1994.0)
suggests.

Unfortunately (but very sensibly) fritzbox checks the dns record of the host to
be updated before it actually calls the service. This makes perfect sense:
Updates are only performed when neccessary. But in our case, the value of the
hostname field is not an existing host, it's the tunnel id. The alternative
hostnames (tunnel#.tunnelbroker.net, user-#.tunnel.tserv#.loc#.ipv6.he.net)
also do not resolve. This results in the following log message on your fritzbox:

        Dynamic DNS error: Dynamic DNS update is disabled until the dynamic DNS registration data are changed.
        Dynamic DNS error: The specified domain name cannot be resolved.

The Solution
============


he.net offers another cool feature: If you host a domain or subdomain on their
free dns service at [dns.he.net](https://dns.he.net/), you can enable dynamic
updates for a hostname and have your tunnel endpoint updates set the dns record
for that hostname, too! Let's walk through this:

You can either:

* move the dns of the whole domain to dns.he.net by updating the NS-records at your current provider/registrar, or:

* use a subdomain and delegate it to dns.he.net. Just create NS records for
something like ``home.example.com`` pointing to ns2.he.net, ns3.he.net, ns4.he.net and ns5.he.net.

Either way, you should now have a (sub-)domain that you can control via
dns.he.net. Create a host in that zone with any ip address and select __enable
entry for dynamic dns__.


{% img left /images/blog/bkw/dns-henet-dynamic.png 605 325 'dns.he.net dynamic host update gui' 'dns.he.net gui screenshot' %}

Next, in the listing of your domain's records, click the refresh symbol in this host's DDNS column:

{% img left /images/blog/bkw/dns-henet-dynamic-key.png 843 333 'dns.he.net dynamic host update gui' 'dns.he.net gui screenshot' %}

Generate a key to use for updates to this host:

{% img left /images/blog/bkw/dns-henet-dynamic-key2.png 606 288 'dns.he.net dynamic host update gui' 'dns.he.net gui screenshot' %}

Save that key, you'll need it in a minute.

Now head over to [tunnelbroker.net](https://tunnelbroker.net), open up your
tunnel's details and select the __Advanced__ tab. Enter the dynamic hostname you configured in the last step and the key you generated.


{% img left /images/blog/bkw/tunnelbroker-dynamic-host.png 606 288 'tunnelbroker dynamic host update gui' 'tunnelbroker.net gui screenshot' %}

Now, whenever the tunnel endpoint is updated, the hostname _gw.home.example.com_ will be updated to your router's ipv4 address as well.

__This allows us to use a valid and up-to-date host in the dynamic dns updater
of the fritzbox.__

Head over to your fritzbox GUI.

Open Internet/Account Information/IPv6, check _IPv6 support enabled_ and
_Always use a tunnel protocol for the IPv6 connection_. Select tunnel protocol
_6in4_ in _Connection Settings_, fill in the values from your tunnel details
page at tunnelbroker.net and hit _Apply_.

Now for the hack:
-----------------

Open Internet/Permit Access/Dynamic DNS, check _Use dynamic DNS_ and select
_User-defined_ for _Dynamic DNS provider_.

{% img left /images/blog/bkw/fritzbox-dyndns.png 606 288 'fritzbox dyndns gui' 'fritzbox gui screenshot' %}

Enter the following URL for _Update URL_, replacing _123456_ with the id of
your he.net tunnel:

    http://ipv4.tunnelbroker.net/nic/update?hostname=123456&username=<username>&password=<pass>

Do not replace the placeholders in angle brackets, your fritzbox will do that.

Enter the dynamic host you configured earlier in the _Domain name_ field.
This will be used by the fritzbox to check whether the update needs to be
performed.

Enter your he.net username and password in the respective fields.

Hit _Apply_ and enjoy an auto updating IPv6 tunnel to hurricane electric.

How does this work?
-------------------

The dynamic hostname (which is kept in sync by hurricane electric) is used for
checking for changes, but the actual update is performed on the tunnel id hardcoded in the _Update URL_.

Bonus
-----

As a useful side effect, from now on you can always reach your router and any
port-forwared services under you own hostname _(gw.home.example.com)_ in your
very own domain!  How cool is that?

Actually, now that you have IPv6, you don't really need those
pesky port forwards any more: just assign any v6 address you like and use
plain internet routing the way it was supposed to work in the first place.


