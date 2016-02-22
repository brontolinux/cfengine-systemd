# cfengine-systemd

**This is still experimental!** I decided to write these unit files as an
exercise while learning systemd. I tried them on my own laptop running
Debian Jessie 8.2 and they work like a charm, but they definitely need
more testing. You are welcome to test them on different distributions,
report success/failure or submit pull requests to fix bugs.


## Features

This distribution provides service files for the three CFEngine daemons (in the community
edition): cf-serverd, cf-execd and cf-monitord. `Restart=always` is
set for the three daemons, so that if any of them
dies for any reason, systemd will start a new instance nearly immediately.

This distribution also provides a special "target", `cfengine3.target`
that is designed to replace
the service `cfengine3.service` automatically created by systemd from
the init.d file. The target file requests (but **doesn't require!**)
the three daemon services to start, so that it's still possible to disable one
or more daemons without causing the target to fail. It also provides means
to stop all the services at once by issuing `systemctl stop cfengine3.target`,
the same way you would stop all services using the init script.



## Installation

The following instructions assume
that you have installed cfengine-community from the [official package
for Debian](https://cfengine.com/product/community/) and that you'll run
these commands as the root user.

1. copy/clone the contents of this repository
2. fully disable the cfengine3 service, built automatically by systemd from
the init.d file:<br/>```systemctl mask cfengine3.service```
3. copy the unit files from the repository into `/etc/systemd/system`:<br/>```cp *.service cfengine3.target /etc/systemd/system```<br/>(but see the note below)
4. reload the configuration of systemd via ```systemctl daemon-reload```
5. now stop the running CFEngine services and start the target, e.g.:<br/>```/etc/init.d/cfengine3 stop ; systemctl start cfengine3.target```

If everything went well, you should like the output of the command `systemctl status cfengine3.target cf-serverd cf-execd cf-monitord`. And if you kill one of the daemons and run the same command again you will like the output even more `:-)`

**Note:** In Debian you also have the option to use
`/usr/local/lib/systemd/system`, as reported in Debian's
[man systemd](http://manpages.debian.org/cgi-bin/man.cgi?query=systemd&apropos=0&sektion=0&manpath=Debian+8+jessie&format=html&locale=en).
I actually used
that directory in my experiments since I didn't want to "pollute"
`/etc/systemd/system`, which contained other stuff created by systemd itself.
You should use `/etc/systemd/system` for cross-distribution compatibility,
but if you are on Debian and you're just experimenting,
`/usr/local/lib/systemd/system` is also a viable choice.


## Disabling services
I have observed a behavior that I didn't expect, probably due to the service
units being called by the target unit instead of directly. If you disable a
service via systemctl, say cf-serverd, that service will be nonetheless
brought up at the next start of the cfengine3.target. To ensure that the
service is really disabled and for good, you need to **mask** it:


```
root@murray:~# systemctl mask cf-serverd.service 
Created symlink from /etc/systemd/system/cf-serverd.service to /dev/null.
root@murray:~# 

```

Notice that, as explained earlier in this document, that won't prevent the
other services to start.
