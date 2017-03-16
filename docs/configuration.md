#Accessing the configuration
There are many ways to access files on a remote host. As we are using a MacBook as our main development environment, here are the steps to make your remote host files accessible from your mac using Apple Filing Protocol.

```bash
sudo apt-get install netatalk
```

Edit the configuration file

```bash
sudo nano /etc/default/netatalk
```

and ensure the below are set correctly  :

```
ATALKD_RUN=no
PAPD_RUN=no
CNID_METAD_RUN=yes
AFPD_RUN=yes
TIMELORD_RUN=no
A2BOOT_RUN=no
```
Save and exit (Ctrl+0 , Ctrl+x)

Then we need to specify which files we are going to share :

```bash
sudo nano /etc/netatalk/AppleVolumes.default
```
at the very end of the file add the below :

```
/your/path  VolumeName allow:@groupname options:usedots,upriv
```
Save and exit (Ctrl+0 , Ctrl+x)

So this could look like the below if you have used hassbian and Home Assistant is already installed:

```
/home/homeassistant/.homeassistant  HomeAssistant allow:@groupname options:usedots,upriv
```

now restart the daemon :

```bash
sudo /etc/init.d/netatalk restart
```

Theoretically, you should now be able to connect to your server from your MacBook. However this would require specifying the URI with the server hostname or IP address. Using Avahi, your server can advertise itself and automagically appear in your Finder.

```bash
sudo apt-get install avahi-daemon libnss-mdns
```

Edit the configuration

```bash
sudo nano /etc/nsswitch.conf
```

Modify the "hosts" line so that it looks like this:

```
hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4 mdns
```

Save and exit (Ctrl+0 , Ctrl+x)

Create a service file for the afp service

```bash
sudo nano /etc/avahi/services/afpd.service
```

and add

```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
    <name replace-wildcards="yes">%h</name>
    <service>
        <type>_afpovertcp._tcp</type>
        <port>548</port>
    </service>
    <service>
        <type>_device-info._tcp</type>
        <port>0</port>
        <txt-record>model=MacPro</txt-record>
    </service>
</service-group>
```
Save, exit (Ctrl+0 , Ctrl+x)

and restart the daemon :

```bash
sudo /etc/init.d/avahi-daemon restart
```

You should now see your server from your OSX Finder.
