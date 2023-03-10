Thanks to: https://klipper.discourse.group/t/how-to-klipperscreen-on-android-smart-phones/1196/40?page=2

**Step 1** 
ssh into your pi with putty or linux command line ssh

**Step 2**
plug your android phone with usb debugging enabled into your raspberry

**Step 3**
if you have not yet installed it install adb
```
sudo apt-get install android-tools-adb
```
**Step 4**
use command, to see if your phone is recognized by adb
```
adb devices
```
List of devices attached

0b4dca7f	device

**Step 5**
install x11-apps
``` 
sudo apt-get install x11-apps
```

**Step 6**
enable port forwarding
```
adb forward tcp:6100 tcp:6000
```

**Step 7**
start x111 program xeyes
```
DISPLAY=:100 xeyes
```

Your telephone should look like this:

![alt text](https://github.com/PrintStructor/VORON-V0.1/blob/main/4516da680f86b680702db926a6a10b90a91d3efa_2_666x500.jpeg?raw=true)

Mind the xeyes in the upper left corner.

If this does not work try if you have to confirm usb debugging on your phone.

You can use ctrl-c to stop xeyes.

If this runs you can proceed to the next step

**Step 8**
Now its time to setup the Script as a system service that survives throughout reboots
look at example.service in data folder that is word for work what you need in your file i have it listed as i am going to be compiling an autoscript

```
sudo nano /etc/systemd/system/KlipperScreen.service
```
paste in nano
```
[Unit]
Description=KlipperScreen
After=moonraker.service
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=pi
WorkingDirectory=/home/biqu/KlipperScreen
ExecStart=/home/biqu/KlipperScreen/launch_klipperscreen.sh

[Install]
WantedBy=multi-user.target
```
Save Buffer by "Ctrl X" then yes

**Step 9**
Time to create the loading script to forward the display to XSDL. I'm doing this in putty to ssh to my CB1
```
sudo nano /home/biqu/KlipperScreen/launch_klipperscreen.sh
```
paste this code into your nano screen


```
#!/bin/bash
# forward local display :100 to remote display :0
adb forward tcp:6100 tcp:6000

adb shell dumpsys nfc | grep 'mScreenState=' | grep OFF_LOCKED > /dev/null 2>&1
if [ $? -lt 1 ]
then
    echo "Screen is OFF and Locked. Turning screen on..."
    adb shell input keyevent 26
fi

adb shell dumpsys nfc | grep 'mScreenState=' | grep ON_LOCKED> /dev/null 2>&1
if [ $? -lt 1 ]
then
    echo "Screen is Locked. Unlocking..."
    adb shell input keyevent 82
fi

# start xsdl
adb shell am start-activity x.org.server/.MainActivity

ret=1
timeout=0
echo -n "Waiting for x-server to be ready "
while [ $ret -gt 0 ] && [ $timeout -lt 60 ]
do
    xset -display :100 -q > /dev/null 2>&1
    ret=$?
    timeout=$( expr $timeout + 1 )
    echo -n "." 
    sleep 1
done
echo ""
if [ $timeout -lt 60 ]
then
    DISPLAY=:100 /home/biqu/.KlipperScreen-env/bin/python3 /home/biqu/KlipperScreen/screen.py
    exit 0
else
    exit 1
fi
```
Save Buffer by "Ctrl X" then yes

**Step 10**
make the script executable
```
chmod a+x /home/biqu/KlipperScreen/launch_klipperscreen.sh
```

**Step 11**
time to enable and reload services 
```
systemctl enable KlipperScreen.service 
```

==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: ,,, (biqu)
Password: 
==== AUTHENTICATION COMPLETE ===
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: ,,, (biqu)
Password: 
==== AUTHENTICATION COMPLETE ===

and start systemd service
```
systemctl start KlipperScreen.service 
```

==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to start 'KlipperScreen.service'.
Authenticating as: ,,, (biqu)
Password: 
==== AUTHENTICATION COMPLETE ===
