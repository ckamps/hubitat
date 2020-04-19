# Alexa Cookie NodeJS wrapper for Hubitat
This webservice is a wrapper of orginal [Apollon77 alexa-cookie library](https://github.com/Apollon77/alexa-cookie) for use with Hubitat [Alexa TTS App](https://github.com/ogiewon/Hubitat/tree/master/Alexa%20TTS).

# How to install
(Tested on Ubuntu 16.04, commands may be a little different on other platforms)
- Deploy a NodeJS server on your preferred platform (plenty of instuctions on web if needed)
- It can be deployed on a cloud service as well, in that case it's suggested to configure authentication and don't publish the service but use it behind a HTTPS reverse proxy
- Download latest version as zip and then unzip it on the server
- Go to folder of AlexaCookieNodeJs
- Make ```AlexaCookie.js``` executable (needed on Unix platforms only) => ```chmod +x ./AlexaCookie.js```
- Execute the command ```npm install``` to install all required npm packages

# Configuration
In case default ports (wrapper on 81 and proxy on 82) have to be changed or to add authentication, modify the file ```config.json```
Empty username and empty password means no authentication required
```
{
	"port":"81",
	"proxyPort":"82",
	"username": "",
	"password": "",
	"consoleLogging": false
}
```

# How to automatically start the wrapper

## Using `pm2`

- Execute command ```npm install -g pm2``` to install Node Process Manager 2 and allow autostart
- Start the application with ```pm2 start AlexaCookie.js``` command
- Execute command ```pm2 startup```
- Execute command ```pm2 save```
- Due to an issue still under investigation, looks like AlexaCookie.js must be restarted after getting succesfully 1 cookie, otherwise the second refresh won't work. To workaround it, schedule ```pm2 restart AlexaCookie.js``` with cron every day

## Using `launchd` on macOS

The following instructions have been tested on macOS Catalina.

1. Copy the example `.plist` files from `macos/` to `/Library/LaunchDaemons/`.
2. Ensure their ownership is set apppropriately:

```
$ cd /Library/LaunchDaemons
$ sudo chown root:wheel org.alexa.cookie.*.plist
```

3. In `org.alexa.cookie.nodejs.plist`, change the `WorkingDirectory` key if you unzipped the archive in a location other than `/Applications/`.
4. Load the two `.plist` files:

```
$ sudo launchctl load ./org.alexa.cookie.nodejs.plist

$ sudo launchctl load ./org.alexa.cookie.nodejs.restart.plist
```
These commands will start the daemon and schedule a restart of the daemon to occur at 10 minutes past midnight. See the `pm2` section above about the known issue for which a workaround is to restart the daemon on a daily basis.

### Restarting the Daemon

The restart `.plist` file executes the following command on a regular schedule, but you can execute it directly to restart the daemon:

```
$ sudo launchctl kickstart -k system/alexa.cookie.nodejs
```

### Troubleshooting `launchd` Issues

If you cannot access the web application on the configured ports, try these troubleshooting steps:

1. See if the daemon is running:

```
$ ps -ef | grep AlexaCookie
```

2. Check the system log:

```
$ sudo tail -f /var/log/system.log
```

3. Check the application's `stderr` and `stdout`:

```
$ sudo tail -f  /var/log/alexa-cookie-nodejs/stderr.log

$ sudo tail -f  /var/log/alexa-cookie-nodejs/stdout.log
```
You can stop the daemon and cancel the scheduled job by executing:

```
$ sudo launchctl unload ./org.alexa.cookie.nodejs.plist

$ sudo launchctl unload ./org.alexa.cookie.nodejs.restart.plist
```

# Usage
- Go to ```http://[serverip]:81```
- Fill the form wih your Amazon username, password and country
- At this point, depending on Amazon security, cookie refresh options could be immediately generated or a message will ask to go to webpage ```http://[serverip]:82``` (proxy) that will be opened in a new tab
- **IMPORTANT!** do not close the original (let's call it :81) tab
- Login again on second page (with 2FA if enabled)
- **IMPORTANT!** After succesfull login on second webpage, a message will be prompted to close the browser: don't close the browser, but instead close only that tab (:82) and go back to the original tab (:81)
- Copy the RefreshURL and RefreshOptions in the relative fields on Hubitat Alexa TTS App
- Refresh will be now handled automatically between Hubitat and wrapper every 6 days, without human intervention requirement
