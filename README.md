# cfengine-systemd

**This is still experimental!**. I decided to write these unit files as an
exercise while learning systemd. I tried them on my own laptop running
Debian Jessie 8.2 and they work like a charm, but they definitely need
more testing. You are welcome to test them on different distributions,
report success/failure or submit pull requests to fix bugs.


## Features

Provides service files for the three CFEngine daemons (in the community
edition): cf-serverd, cf-execd and cf-monitord. It also provides a special
"target" that requests, but **doesn't require**, the three services to
start, so that it's still possible to disable one or more daemons without
causing the target to fail.

`Restart=always` is also set for the three daemons, so that if any of them
dies, for any reason, systemd will start a new instance nearly immediately.

## Installation

The following instructions assume
that you have installed cfengine-community from the [official package
for Debian](https://cfengine.com/product/community/) and that you'll run
these commands as the root user.

1. copy/clone the contents of this repository
2. fully disable the cfengine3 service, built automatically by systemd from
the init.d file: ```systemctl mask cfengine3.service```
3. copy the unit files from the repository into `/usr/local/lib/systemd/system`: ```cp *.service cfengine3.target /usr/local/lib/systemd/system```
4. reload the configuration of systemd via ```systemctl daemon-reload```
5. now stop the running CFEngine services and start the target, e.g.: ```/etc/init.d/cfengine3 stop ; systemctl start cfengine3.target```

If everything went well, you should like the output of the command `systemctl status cfengine3.target cf-serverd cf-execd cf-monitord`. And if you kill one of the daemons and run `systemctl` again you will like the output even more `:-)`

