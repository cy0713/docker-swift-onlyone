#Docker OpenStack Swift onlyone

Simple deployment of a "all in one" style OpenStack Swift server, uses Ubuntu packages as opposed to source.

## Makefile and swiftrc

There is a makefile with some useful helper commands, as well as a swiftrc file that you can use as well which shorts the swift command as well as adds a function to set a container to be a public, listable html page.

## Usage

I suggest using the data container methodology.

So first we create a data only container for /srv.

```bash
**pull busybox first**：$ docker pull busybox
vagrant@host1:~$ docker run -v /srv --name SWIFT_DATA busybox
```
Note: busybox是一个聚成了一百多个最常用linux命令和工具的软件工具箱，它在单一的可执行文件中提供了精简的Unix工具集。BusyBox可运行于多款POSIX环境的操作系统中，如 Linux(包括Android),hurd,freebsa等，busybox既包含了一些简单实用工具，如cat 和 echo ,也包含了一些更大，更复杂的工具，如 grep ,find ,mount 以及telnet.


Now that we have a data container, we can use the "--volumes-from" option when creating the "onlyone" container. Note that in this case I've called the image built from this docker file "curtis/swift-onlyone".

```bash
**build image first**：$ docker build -t swift-onlyone .
vagrant@host1:~$ ID=$(docker run --name onlyone --hostname onlyone -d -p 12345:8080 --volumes-from SWIFT_DATA -t swift-onlyone)
```

With that container running we can now check the logs.

```bash
vagrant@host1:~$ docker logs $ID
Device d0r1z1-127.0.0.1:6010R127.0.0.1:6010/sdb1_"" with 1.0 weight got id 0
Reassigned 128 (100.00%) partitions. Balance is now 0.00.
Device d0r1z1-127.0.0.1:6011R127.0.0.1:6011/sdb1_"" with 1.0 weight got id 0
Reassigned 128 (100.00%) partitions. Balance is now 0.00.
Device d0r1z1-127.0.0.1:6012R127.0.0.1:6012/sdb1_"" with 1.0 weight got id 0
Reassigned 128 (100.00%) partitions. Balance is now 0.00.
WARNING: Unable to modify file descriptor limit.  Running as non-root?
Starting proxy-server...(/etc/swift/proxy-server.conf)
Starting container-server...(/etc/swift/container-server.conf)
Starting account-server...(/etc/swift/account-server.conf)
Starting object-server...(/etc/swift/object-server.conf)
WARNING: Unable to modify file descriptor limit.  Running as non-root?
Starting container-updater...(/etc/swift/container-server.conf)
Starting account-auditor...(/etc/swift/account-server.conf)
Starting object-replicator...(/etc/swift/object-server.conf)
Starting container-replicator...(/etc/swift/container-server.conf)
Starting object-auditor...(/etc/swift/object-server.conf)
Unable to locate config for object-expirer
Starting container-auditor...(/etc/swift/container-server.conf)
Starting account-replicator...(/etc/swift/account-server.conf)
Starting account-reaper...(/etc/swift/account-server.conf)
Starting container-sync...(/etc/swift/container-server.conf)
Starting object-updater...(/etc/swift/object-server.conf)
Starting to tail /var/log/syslog...(hit ctrl-c if you are starting the container in a bash shell)
```

At this point OpenStack Swift is running.

```bash
vagrant@host1:~$ docker ps
CONTAINER ID        IMAGE                         COMMAND                CREATED             STATUS              PORTS                     NAMES
4941f8cd8b48        curtis/swift-onlyone:latest   /bin/sh -c /usr/loca   58 seconds ago      Up 57 seconds       0.0.0.0:12345->8080/tcp   hopeful_brattain
```

We can now use the swift python client to access Swift using the Docker forwarded port, in this example port 12345.

```bash
**install python-swiftclient first**：$ sudo apt install python-swiftclient
vagrant@host1:~$ swift -A http://127.0.0.1:12345/auth/v1.0 -U test:tester -K testing stat
       Account: AUTH_test
    Containers: 0
       Objects: 0
         Bytes: 0
  Content-Type: text/plain; charset=utf-8
   X-Timestamp: 1402463864.77057
    X-Trans-Id: tx4e7861ebab8244c09dad9-005397e678
X-Put-Timestamp: 1402463864.77057
```

Try uploading a file:

```bash
**prepare file to upload first**：$ sudo vim swift.txt
vagrant@host1:~$ swift -A http://127.0.0.1:12345/auth/v1.0 -U test:tester -K testing upload swift swift.txt
swift.txt
```

That's it!

## Todo

* It seems supervisord running as root in the container, a better way to do this?
* bash command to start rsyslog is still running...
* Add all the files in /etc/swift with one ADD command?
* supervisor pid file is getting setup in /etc/
