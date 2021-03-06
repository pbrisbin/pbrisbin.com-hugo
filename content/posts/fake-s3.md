---
title: Fake S3
date: 2012-10-23
tags: [ruby]
---

[Fake S3][] is a gem designed to run a small server on `localhost` that 
will correctly handle AWS/S3 GETs and PUTs (among others) while using 
the local filesystem as the storage backend. This has a number of 
benefits for anyone working on an application with AWS integration.

Probably the biggest benefit is offline development. There's also a 
lower configuration burden as there's no longer special bucket names or 
keys the manage, the development environment just expects `localhost` to 
work. Finally, there's a very real cost savings. Originally, we paid for 
two buckets *per developer* just to support the sample data for dev and 
test. Now we need none.

[Fake S3]: https://github.com/jubos/fake-s3

## Install

```
$ gem install fakes3
```

## Run

```
$ fakes3 -r /mnt/s3 -p 4567
```

## Configure

For example:

```yaml 
development:
  bucket_name: development

  # doesn't matter
  access_key_id:     123
  secret_access_key: abc

  # does matter
  server: localhost
  port:   4567
```

Assuming you've got some rake tasks which load up S3 with sample data, 
just go ahead and run them, you'll find files created directly under 
`/mnt/s3/development` and the requests your local instance makes will 
just return that data.

All with no cloud-access needed.

## Init script

In case you didn't notice, the `fakes3` command is a bit verbose, which 
may be OK, but if you're like me and just want to run this as a 
background service, the following (naive) init script should do the job:

```bash 
#!/bin/bash

if [[ ! -d /mnt/s3 ]]; then
  mkdir /mnt/s3 || return 1
  chown -R vagrant:vagrant /mnt/s3 # make it user-writable
fi

case $1 in
  start)
    fakes3 -r /mnt/s3 -p 4567 &>/dev/null &
    echo $! > /var/run/fakes3.pid
    ;;
  stop)
    kill -9 `cat /var/run/fakes3.pid`
    ;;
  restart)
    $0 stop
    sleep 3
    $0 start
    ;;
esac
```
