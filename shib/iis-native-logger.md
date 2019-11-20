# Configuring native.log for Shibboleth SP3 on IIS

One of the complaints we receive the most from clients utilizing Shibboleth Service Provider (SP) on Windows Server / IIS, is that `native.log` is empty or missing.

This log, while under typical operational loads is generally indistinguishable in its content from `shibd.log`, the native logger provides valuable information about internal request mapper routing. This can vary from unimportant, to relevant, to absolutely critical information when it comes to diagnosing SP issues.

On Windows systems, the default native logging is to the Windows Event Log, which is observable with the Event Viewer (under `Application and Services Logs -> Shibboleth`).

The following configuration snippet is taken from the distributed `native.logger` file, and configures the log4j (logging provider for Shibboleth) appender for th Event Viewer:

~~~~
log4j.appender.native_log=org.apache.log4j.NTEventLogAppender
log4j.appender.native_log.source=Shibboleth Service Provider
log4j.appender.native_log.layout=org.apache.log4j.PatternLayout
log4j.appender.native_log.layout.ConversionPattern=%c %x: %m
~~~~

`org.apache.log4j.NTEventLogAppender` triggers the logging to the Windows Event log. You can adjust the other parameters of this logging here if you are content with this behavior.

To get more 'natural' behavior, you can instead define the following appender in `native.log`

~~~~
log4j.appender.native_log=org.apache.log4j.RollingFileAppender
log4j.appender.native_log.fileName=C:/opt/shibboleth-sp/var/log/shibboleth/native.log
log4j.appender.native_log.maxFileSize=1000000
log4j.appender.native_log.maxBackupIndex=10
log4j.appender.native_log.layout=org.apache.log4j.PatternLayout
log4j.appender.native_log.layout.ConversionPattern=%d{%Y-%m-%d %H:%M:%S} %p %c %x: %m%n
~~~~

and Shibboleth will write this information into the `native.log` file of your specification. This is the default behavior on Linux-based installs, but isn't done on Windows because this alone is insufficient. You must also ensure that the file you want Shibboleth to write to is given writable by the IIS service account, `IUSR`:

<div style="text-align: center;"><img src="https://idmengineering.com/screenshots/2019-11-20%2010_01_16.png"></div>

The above screenshot demonstrates the permissions that we utilize on one of our dev IIS servers which is writing logs to `native.log`.

---

Need help troubleshooting Shib SP on IIS? [Give us a call!](https://idmengineering.com/contact) We'd be glad to help! 
