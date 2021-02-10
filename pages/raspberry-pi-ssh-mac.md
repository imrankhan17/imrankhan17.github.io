## Connecting to a Raspberry Pi via SSH on Mac

On your Mac download and install the [Raspberry Pi Imager](https://www.raspberrypi.org/software/) application.

Insert a SD card or USB drive into you Mac.  Select "Raspberry Pi OS Lite" and click "WRITE".

Once the OS is downloaded:

```bash
cd /Volumes/boot
touch wpa_supplicant.conf
```

Insert the following into `wpa_supplicant.conf`:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
 ssid="wifi network name"
 psk="wifi password"
}
```

[More info](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)

To enable SSH access to the Pi:
```
cd /Volumes/boot
touch ssh
```

[More info](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md)

Remove the SD card/USB drive, insert it into the Pi and power it up.

After a few minutes, you can SSH into the Pi with:

```bash
ssh -p 22 pi@raspberrypi.local
```

The default password is `raspberry`.

Once SSH'd, change the password with `passwd`.

To avoid entering a password every time, copy the contents of a public key on your Mac e.g. `~/.ssh/id_rsa.pub`, and paste it into a file called `~/.ssh/authorized_keys` on the Pi.

On your Mac, add the following entry to your `~/.ssh/config` file:

```
Host pi
        Hostname raspberrypi.local
        IdentityFile ~/.ssh/id_rsa
        User pi
        Port 22
```

You can now SSH into the device with:

```bash
ssh pi
```

#### Additional set-up

```bash
sudo su -
adduser imran
usermod -a -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,gpio,i2c,spi imran

# exit and run `ssh -p 22 imran@raspberrypi.local`
sudo su -
pkill -u pi
deluser -remove-home pi
```

```bash
sudo su -
visudo /etc/sudoers.d/010_pi-nopasswd

# replace with `imran ALL=(ALL) PASSWD: ALL`
```

```bash
mkdir /home/imran/.ssh
vi /home/imran/.ssh/authorized_keys
# copy contents of ~/.ssh/id_rsa.pub
```

```bash
sudo su -
vi /etc/ssh/sshd_config
```

```
AllowUsers imran
Port 2022

ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```

```bash
systemctl restart ssh
```

```bash
sudo su -
hostnamectl set-hostname 'mypi'
shutdown -r now
```

```bash
touch .bash_aliases
echo "alias ll='ls -laht'" >> .bash_aliases

sudo su -
cp /home/imran/.bashrc ~
cp /home/imran/.bash_aliases ~
```

---
[Home](../index.md)
