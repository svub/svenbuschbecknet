---
layout: post
title:  "Laptop wakes up during sleep"
date:   2019-08-02 21:36:14 +0800
categories: linux fedora
---
# Laptop wakes up during sleep

Running Fedora 29, my laptop had sleeping issues. It just kept waking up during sleep.
Poor laptop.
Checked for wake-on-lan WOL > nope.
Checked for bluetooth > suspend with bluetooth headset on > turn it off > laptop wakes up > bingo!

I'm reproducing
[the brilliant solution form the Manjaro forum](https://forum.manjaro.org/t/laptop-wakes-from-sleep-if-bluetooth-is-enabled/50647)
here so you don't have to click.
But please give kudos on the forum!

Create `/etc/systemd/system/bluetooth-reload.service` -- e.g. with `sudo vim` -- and put:

```bash
[Unit]
Description=Reload bluetooth driver after system resume
After=hibernate.target suspend.target hybrid-sleep.target

[Service]
Type=oneshot
TimeoutSec=15
ExecStart=/usr/sbin/modprobe btusb

[Install]
WantedBy=hibernate.target suspend.target hybrid-sleep.target
```

Same for `/etc/systemd/system/bluetooth-suspend.service`:

```bash
[Unit]
Description=Bluetooth module suspend helper
Before=sleep.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target

[Service]
Type=simple
ExecStart=-/usr/bin/bluetooth-suspend.sh

[Install]
WantedBy=sleep.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target
```

And `/usr/bin/bluetooth-suspend.sh`:

```bash
#!/bin/bash
modprobe -r btusb
```

Finally, make the files executable, set ownership to root and enable the service:

```bash
sudo chmod +x /etc/systemd/system/bluetooth-reload.service
sudo chown root:root /etc/systemd/system/bluetooth-reload.service
sudo chmod +x /etc/systemd/system/bluetooth-suspend.service
sudo chown root:root /etc/systemd/system/bluetooth-suspend.service
sudo chmod +x /usr/bin/bluetooth-suspend.sh
sudo chown root:root /usr/bin/bluetooth-suspend.sh
sudo systemctl enable bluetooth-suspend.service
sudo systemctl start bluetooth-suspend.service
sudo systemctl enable bluetooth-reload.service
sudo systemctl start bluetooth-reload.service
```

You can test it the same way as before. No reboot needed for me. :)

# Bonus: find modules

Next thing that can to my mind is to disable WiFi during sleep.
Just draining the battery and I don't need it.
How to find the module?
There are two options, `lsusb` and `lspci`,
listing USB and PCI devices.
If you're not sure with is yours, try both and see if you get something useful.
Fore example, `lspci -vvnn | grep -A 9 Network` lists the kernel module being used for may WiFi.
