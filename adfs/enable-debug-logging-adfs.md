# Enabling 'Debug' Logging in ADFS

Microsoft Active Directory Federation Services (ADFS) isn't the simplest SAML implementation to debug. When a new service provider ("relying party") integration isn't working, when configuring a new identity provider ("claims provider"), or just having issues with a particular user accessing a service, there is often little-to-no useful information within the default logs. That said, it's relatively simple to lower the logging level:

## Set Trace level and Enable the ADFS Tracing Log

1. Run command prompt as an administrator.
2. Type the following command: `wevtutil set-log “AD FS Tracing/Debug” /L:5`
3. Open Event Viewer.
4. Right-click on Application and Services Logs.
5. Select View -> "Show Analytics and Debug Logs"
6. Navigate to Applications and Services Logs -> AD FS Tracing –> Debug.
7. Right-click and select "Enable Log" to start trace debugging immediately.

### To stop tracing, similarly:

1. Follow Steps 1-6 above.
2. Right-click and select "Disable Log" to stop trace debugging. It is difficult to scroll and search in the events page by page in the debug log, so it is recommended that you save all debug events to a \*.evtx file first.
3. Open the saved log again and observe that it now includes ADFS Tracing events.

> Note: Trace/Debug logs in ADFS are *very* chatty... and should be used with discretion, and only for the duration of troubleshooting activity, on production servers.

## Enable Object Access Auditing to See Access Data

To observe detailed information about access activities on the ADFS servers you must enable object access auditing in two locations on the ADFS servers:

### To Enable Auditing:
1. On the primary ADFS server, right-click on Service.
2. Select the Success audits and Failure audits checkboxes. These settings are valid for all ADFS servers in the farm.

### To modify the Local Security Policy, do the following:

1. Right-click the Start Menu, and select 'Run'
2. Type gpedit.msc and select 'OK'
3. Navigate to Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Audit Policy
4. In the policy list, right-click on Audit Object Access, and select 'Properties'
5. Select the Success and Failure checkboxes. These settings have to be enabled in the Local Security Policy on each ADFS server (or in an equivalent GPO that is set in Active Directory).
6. Click OK

Open the security event logs on the ADFS servers and search for the timestamps that correspond to any testing or troubleshooting that is being conducted.
