# Enterprise Console Service for AppDynamics on Linux Platform
This script can be copied in Linux with modifications to run the Enterprise Console as Linux Service. To ensure that Enterprise Console keeps on running in case of server restart.

## Pre-requisites
Make sure that enterprise console is installed on the linux server

## Steps
1. Copy the below script to /etc/init.d on your Linux. Provide permission to execute. use sudo permissions if required and modify the ownership to the same user as the 
```
cp platform-admin-service /etc/init.d/
chmod +x platform-admin-service
```
2. Modify the init info or accept default as per your architecture. The following variables should be looked into
HOME - Path to the enterprise console platform admin home directory
RUNAS - User account which is used to install and run enterprise console
PORT - https/http port number of enterprise console

```
#!/bin/sh

### BEGIN INIT INFO
# Provides:          platform-admin-service
# Required-Start:    $local_fs $network $named $time $syslog
# Required-Stop:     $local_fs $network $named $time $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       This is an AppDynamics enteprise Console Service for Linux.
### END INIT INFO

HOME=/apm/appdynamics/platform/platform-admin
RUNAS=appadmin

PIDFILE=$HOME/platform-admin.pid
PORT=9191

start()
{
	if [ -f "$PIDFILE" ] && kill -0 $(cat "$PIDFILE"); then
		echo 'Service already running' >&2
		return 1
	fi
	echo 'Starting service…' >&2
	$HOME/bin/platform-admin.sh start-platform-admin
	echo 'Service started' >&2
}

stop()
{
if [ ! -f "$PIDFILE" ] || ! kill -0 $(cat "$PIDFILE"); then
    echo 'Service not running' >&2
    return 1
  fi
  echo 'Stopping service…' >&2
  $HOME/bin/platform-admin.sh stop-platform-admin
  echo 'Service stopped' >&2

}

status()
{
	if [ -f "$PIDFILE" ] && kill -0 $(cat "$PIDFILE"); then
		echo "Ping Enterprise Console Version URI" >&2
		curl -k "https://{$HOSTNAME}:{$PORT}/service/version"
		echo ""
		echo 'Service is running' >&2
		return
	fi
	echo 'Service is stopped' >&2
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  status)
    status
    ;;
  retart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
esac
```
