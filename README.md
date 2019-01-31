# Enterprise Console Service for AppDynamics on Linux Platform
This script can be copied in Linux with little modifications to run the Enterprise Console as Linux Service. To ensure that Enterprise Console keeps on running in case of server restart.

## Pre-requisites
Make sure that enterprise console is installed on the linux server

## Steps
1. Copy the below script to /etc/init.d on your Linux. Provide permission to execute. use sudo permissions to copy if required and then modify the ownership to the linux user that is used to install and run enterprise console
```
cp platform-admin-service /etc/init.d/
chmod +x platform-admin-service
```
2. Modify the init info or accept default as per your architecture. The following variables should be looked into
* HOME - Path to the enterprise console platform admin home directory
* RUNAS - User account which is used to install and run enterprise console
* PORT - https/http port number of enterprise console

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

HOME=/<Path>/appdynamics/platform/platform-admin
RUNAS=<Linux User that has permission to run enterprise console>

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
3. Service Commands
   - Service Status
   ```
   [<UserName>@<Hostname> ~]$ service platform-admin-service status
   Ping Enterprise Console Version URI
	{"version":"4.5.5.16839","build":"16839"}
	Service is running
	[<UserName>@<Hostname> ~]$
   ```
   - Service Stop
   ```
   [<UserName>@<Hostname> ~]$ service platform-admin-service stop
	Stopping service…
	Attempting to stop process with id [30543]...
	.
	***** Enterprise Console application stopped *****
	..
	***** Enterprise Console Database stopped *****
	Service stopped
	[<UserName>@<Hostname> ~]$
   ```
   - Service Start
   ```
   [<UserName>@<Hostname> ~]$ service platform-admin-service start
	Starting service…
	Starting Enterprise Console Database
	..
	***** Enterprise Console Database started *****
	Starting Enterprise Console application
	Waiting for the Enterprise Console application to start.........
	***** Enterprise Console application started on port 9191 *****
	Service started
	[<UserName>@<Hostname> ~]$
   ```
