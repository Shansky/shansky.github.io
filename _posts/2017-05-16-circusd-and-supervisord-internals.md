---
layout: post
title: Supervise internals
---

Sometimes you need somebody to look after your process :)

## supervisor

In python2 world it\'s common tool to use [supervisor](https://github.com/Supervisor/supervisor) to look after your application processes.

Supervisor in version 4.0 will support python 3.

Supervisor consists of daemon - called supervisord and control application - called supervisorctl which talks to daemon over socket.

Some supervisor configuration file looks like that:

```
[supervisord]
logfile = /var/log/python-application/supervisor/supervisord.log
logfile_maxbytes = 10MB
logfile_backups = 10
loglevel = info
pidfile = /tmp/python-application-supervisord.pid
nodaemon = false
minfds = 1024
minprocs = 200

[unix_http_server]
file = /tmp/python-application-supervisord.sock
chmod = 0770

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl = unix:///tmp/python-application-supervisord.sock

[group:pyapps]
programs=gunicorn,celery

[program:gunicorn]
directory = /opt/python-application/src
environment = DJANGO_SETTINGS_MODULE="settings.production"
command = gunicorn wsgi:application -b 0.0.0.0:%(ENV_APPLICATION_PORT)s --reload
stdout_logfile = /var/log/python-application/supervisor/gunicorn.log
stderr_logfile = /var/log/python-application/supervisor/gunicorn.log
autostart = True
autorestart = True
stopsignal = KILL
stopasgroup = true

[program:celery]
directory = /opt/python-application/src
environment = DJANGO_SETTINGS_MODULE="settings.production",C_FORCE_ROOT=true
command = celery -A pyapp worker -c 1 -E
stdout_logfile = /var/log/python-application/supervisor/celery.log
stderr_logfile = /var/log/python-application/supervisor/celery.log
autostart = True
autorestart = True
stopsignal = INT
stopwaitsecs = 60
stopasgroup = true
```

To connect to daemon with that configuration you should run:

```bash
supervisorctl -c configuration_file.ini
```

Now you can start, stop, restart, reload programs or all groups of programs.
When you change configuration file you should run `reread` command to reload configuration on daemon side.

Configuration describes group of processes called *pyapps* which contains two applications or programs: *gunicorn* and *celery* - this names can be different than running command, but this way it's more clear.
Every program has directory with source code, command which supervisor should run when starting application. You can define stdout and stderr log files. You can autostart program when it crashes. You can declare some environment variables (comma separated).
One unusual entry: `%(ENV_APPLICATION_PORT)s` allows you to load environment variable named *APPLICATION_PORT* and substitute it in supervisor configuration file.

## circus

Supervisor currently doesn\'t work with python3 so you need some other tool like [circus](https://github.com/circus-tent/circus) from mozilla to deal with.

Like supervisor, circus has two main parts: daemon - circusd and control application - circusctl.

Some example configuration file for circus:

```
[circus]
debug = True

[watcher:gunicorn]
cmd = gunicorn wsgi:application -t 1800 -b 0.0.0.0:80 --reload --threads=3
numprocesses = 1
working_dir = /opt/python-application/src
stdout_stream.class = FileStream
stdout_stream.filename = /var/log/python-application/circus/gunicorn.log
stdout_stream.refresh_time = 0.5
stderr_stream.class = FileStream
stderr_stream.filename = /var/log/python-application/circus/gunicorn.log
stderr_stream.refresh_time = 0.5
```

To control circus daemon you use `circusctl` command.