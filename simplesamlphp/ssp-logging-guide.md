# Configuring SimpleSAMLphp Logging

Unfortunately, the SimpleSAMLphp documentation is a bit [lacking](https://simplesamlphp.org/docs/stable/simplesamlphp-maintenance#section_4) in this area, so I thought it would be useful to document how to configure the various logging options with SimpleSAMLphp. Since SSP is actively maintained, it's worth noting that this document was prepared with SimpleSAMLphp 1.17.7 which is likely to NOT be the [latest](https://simplesamlphp.org/archive) version available, even though it is at the time this document was created.

<small>Note: All files will be referenced with respect to `$SSP_DIR` which in a typical install is `/var/simplesamlphp`:

## Logging to Files

By default, SimpleSAMLphp comes with the `syslog` facility enabled in `$SSP_DIR/config/config.php` at the `NOTICE` level:

~~~
'logging.level' => SimpleSAML\Logger::NOTICE,
'logging.handler' => 'syslog',
~~~

However, this is not so convenient for practical debugging, where you most likely want to log to files, in this case you should specify:

~~~~
'logging.handler' => 'file',
~~~~

in which case SSP will write logs to the file `simplesamlphp.log` in the directory specified earlier within the same configuration file:

~~~
'loggingdir' => 'log/',
~~~

which defaults to `$SSP_DIR/log`.

> Note: you will want to ensure that any directory that you specify for the `loggingdir` has the proper file system permissions. The service account under which your webserver runs will need read, write, and execute permissions for this directory. Since that account typically already owns the `loggingdir` this is usually just a simple matter of: `chmod u+rwx` if you notice that no logging data is being written to the specified directly.

## Rotation

SimpleSAMLphp won't automatically perform rotation of the `simplesamlphp.log` file... so you must do it manually with a tool such as [**logrotate**](https://www.vultr.com/docs/using-logrotate-to-manage-log-files). For a quick example, after installing `logrotate` if you add the following to `/etc/logrotate.d/simplesamlphp`:

~~~~
/var/simplesamlphp/log/simplesamlphp.log {
  daily
  missingok
  notifempty
  copytruncate
  dateext
  rotate 30
}
~~~~

The logfile `/var/simplesamlphp/log/simplesamlphp.log` will rotate daily for 30 days.
