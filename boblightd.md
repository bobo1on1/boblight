# Introduction #

Boblightd is a daemon that translates light data from clients into commands for external controllers.


# Details #

```
Usage: boblightd [OPTION]

  options:

  -c  set the config file, default is /etc/boblight.conf
  -f  fork
```

Boblightd needs a [config](boblightconf.md) file so it knows what to do at startup, the default file it tries to open is [/etc/boblight.conf](boblightconf.md), this can be changed with the -c flag, for example: "boblightd -c /home/user/boblight.conf".

By default boblightd does not fork, so if you want to start it from a boot script you should use the -f flag.

Boblightd catches SIGTERM and SIGINT, so it can be stopped by pressing ctrl-c in the terminal it was started at (presuming it didn't fork), or by killall, at which point it will kick off all the clients and shut down the devices properly.

It keeps a logfile at "~/.boblight/boblightd.log", any previous logfile will be renamed at startup, 5 old logfiles are kept around, the newest is boblightd.log.old.1 and the oldest is boblightd.log.old.5.

The log is currently also printed to stderr, it looks something like this:

```
(InitLog)                     
(InitLog)                     start of log /home/bob/.boblight/boblightd.log
(main)                        boblight revision: 213:261M
(PrintFlags)                  starting ./boblightd
(CConfig::LoadConfigFromFile) opening /etc/boblight.conf
(CConfig::CheckConfig)        checking config lines
(CConfig::CheckConfig)        config lines valid
(CConfig::BuildConfig)        building config
(CConfig::BuildConfig)        built config successfully
(CConnectionHandler::Process) starting connection handler on *:19333
(CClientsHandler::Process)    starting clients handler
(main)                        starting timers
(main)                        starting devices
(CDevice::Process)            ambilight1: starting with output "/dev/ttyUSB0"
(CDevice::Process)            ambilight1: setting up
(CDeviceLtbl::OpenController) ambilight1: controller opened
(CDevice::Process)            ambilight1: setup succeeded
(CDeviceLtbl::CloseController)ambilight1: controller closed
(SignalHandler)               caught SIGTERM
(main)                        signaling devices to stop
(main)                        signaling connection handler to stop
(main)                        signaling clients handler to stop
(main)                        signaling timers to stop
(main)                        waiting for devices to stop
(CClientsHandler::Process)    disconnecting clients
(CClientsHandler::Process)    clients handler stopped
(CDevice::Process)            ambilight1: closed
(CDevice::Process)            ambilight1: stopped
(main)                        waiting for timers to stop
(main)                        waiting for connection handler to stop
(CConnectionHandler::Process) connection handler stopped
(main)                        waiting for clients handler to stop
(main)                        exiting
```