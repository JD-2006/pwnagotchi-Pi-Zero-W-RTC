Item number: HCRASP0009
Accuracy: ± 5ppm (± 0.432 sec / day) within -40oC to +85oC temperature range.
Battery backup allowing accurate time keep without a power supply
Low power operation
DS3231 controller.
Clock functions include seconds, minutes, hours, day, date, month and year timing.
Timing valid up to year 2100
Two calendar alarm
High-speed (400kHz) I2C serial bu
+2.3 V to +5.5 V supply voltage

$ sudo apt install python3-smbus i2c-tools

$ sudo raspi-config

 Hit enter and then in the next screen select 'I2C'

Answer yes to confirm that you wish to enable I2C support

Select OK on the next prompt and finally when asked if you would like the I2C kernel module loaded by default answer YES. Click OK one final time to confirm.

Once back in the config screen select finish to get back to the command prompt. You should be prompted if you wish to reboot. Select yes.
If you're not asked to reboot then just issue the following command at the command prompt:
$ sudo reboot -h now

Open up another terminal windows and issue the following command:
$ sudo i2cdetect -y 1

We first need to load the RTC module. Issue the following command:
$ sudo modprobe rtc-ds1307

Next switch to a super user with the following command:
$ sudo bash

# echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device

Exit out of super use by typing:

# exit

Reading the current time from the module:
$ sudo hwclock -r

Setting the correct time and date:
Make sure your Pi is connected to the internet and issue the following command:

$ date

ou can simply write this correct time and date to the RTC module with the following command:
$ sudo hwclock -w

Confirm that it has been written connrectly by reading the time out of the RTC module again:
$ sudo hwclock -r

The module should now report the correct time and date.

Configuring your Raspberry Pi to read the RTC module at reboot.

$ sudo nano /etc/modules

add
rtc-ds1307


Next we need to edit the rc.local file which will create the RTC module as a new device when your Pi is booted.
Issue the following command to edit the rc.local file:

$ sudo nano /etc/rc.local

In the text edit add the follwing lines to the bottom of the file, but before the 'exit 0' line:

sleep 15

bash -c echo ds1307 0x68 /sys/class/i2c-adapter/i2c-1/new_device
sleep 0.5
hwclock -s
date

$ sudo reboot -h now

$ sudo apt remove fake-hwclock

$ sudo nano /boot/config.txt

Add the following option to the end of it:
dtoverlay=i2c-rtc,ds3231

$ sudo nano /lib/udev/hwclock-set

Comment out the following lines:
#if [ -e /run/systemd/system ] ; then
#   exit 0
#fi

$ sudo rm /etc/cron.hourly/fake-hwclock

Add a new hourly cronjob that syncs up the clocks.
$ sudo touch /etc/cron.hourly/hwclock
$ sudo chmod +x /etc/cron.hourly/hwclock
$ sudo nano /etc/cron.hourly/hwclock

And add the following lines to it:
#!/bin/sh
/sbin/hwclock --systohc

Turns out there is nothing that actually reads the hwclock back to the system on boot.
So I added a systemd unit to do this:
$ sudo nano /etc/systemd/system/hwclock-start.service

[Unit]
Description=hwclock-start to read rtc and write to system clock
After=sysinit.target

[Service]
Type=oneshot
ExecStart=/sbin/hwclock --hctosys --utc

[Install]
WantedBy=basic.target

$ sudo systemctl daemon-reload
$ sudo systemctl enable hwclock-start.service
