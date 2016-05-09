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
A service file for an "umbrella" cfengine3 service is also
provided. Its function is to mask cfengine3.service file provided with
the community edition that makes calls to the /etc/init.d/cfengine3
file. Thee services cf-serverd, cf-exec-d and cf-monitord, when
enabled, are installed as pre-requisites for cfengine3 service, making
systemd to start them when cfengine3 is started.


## Installation

The following instructions assume
that you have installed cfengine-community from the [official package
for Debian](https://cfengine.com/product/community/) and that you'll run
these commands as the root user.

1. copy/clone the contents of this repository
2. stop the running CFEngine services, e.g. via ```/etc/init.d/cfengine3 stop```
3. copy the unit files from the repository into `/etc/systemd/system`:<br/>```cp *.service /etc/systemd/system```<br/>(but see the note below)
4. reload the configuration of systemd via ```systemctl daemon-reload```
5. enable individual services that should be started by cfengine service:<br/>```systemctl enable cf-execd.service```<br/> and same for cf-execd.service and cf-monitord.service
6. start the new cfengine3.service:<br/> ```systemctl start cfengine3.service```
7. to start services during boot enable cfengine3 service via
   ```systemctl enable cfengine3.service```

If everything went well, you should like the output of the command `systemctl status cf-serverd cf-execd cf-monitord`. And if you kill one of the daemons and run the same command again you will like the output even more `:-)`

**Note:** In Debian you also have the option to use
`/usr/local/lib/systemd/system`, as reported in Debian's
[man systemd](http://manpages.debian.org/cgi-bin/man.cgi?query=systemd&apropos=0&sektion=0&manpath=Debian+8+jessie&format=html&locale=en).
I actually used
that directory in my experiments since I didn't want to "pollute"
`/etc/systemd/system`, which contained other stuff created by systemd itself.
You should use `/etc/systemd/system` for cross-distribution compatibility,
but if you are on Debian and you're just experimenting,
`/usr/local/lib/systemd/system` is also a viable choice.

## "Umbrella" service and dependencies

cfengine3.service does not start anything by itself. It is referenced
by other three services in their "RequiredBy=" directives, which will
make systemd create symbolic links to these individual services in the
cfengine3.requires directory (most likely under /etc/systemd/system,
but this might depend on a distribution) if those services are enabled
with ```systemctl enable``` command. This in turn will instruct
systemd to start those services when cfengine3 is started.

Please note that even if individual services are enabled from the
systemd's point of view (```systemctl is-enabled cf-execd``` returns
```enabled```) it will not be started on boot if cfengine3.service is
disabled. On the other hand all components of CFEngine can be enabled
(or disabled) by only enabling (or disabling) cfengine3.service.
