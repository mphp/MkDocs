#Setting up ssh
Now that your Raspberry or PC is installed with Ubuntu, Raspbian or any other Linux flavor, it is time to access it. As it will be used as a server, there is no need to plug a monitor, or to have it around you. It can lie in a closet, next to your router. As long as it is network wired.

SSH is the easiest and most secure way to access your server. You will need access to manage services running there, as well as installing new software depending on your usage. It's also needed so that you can change the default password of the default user, specifically on the Raspberry Pi.


First, find out your server IP address. The easiest way is to connect to your router administration page and from there find your device and its IP address. Your router would have assigned an IP address to your server through DHCP.

While you are there, and because we are setting up a server we expect it's address to remain the same over time, so go and check for the place where you can assign a fix IP local IP address for your server. Once done, take note of that IP address.

Now, from your MacBook, open the Terminal application and type:

```bash
ssh <YourUser>@<YourIPAddress>
```
So that it looks like this:

```bash
ssh pi@192.168.1.10
```

Once you press enter you should be prompted to accept the server key. You are then connected.

If you did not set a password for your current user on the server, then you now need to change the default one. At the command prompt, type :

```bash
passwd
```
and follow the instructions.

Now to prevent having to provide a password each time you connect to your server from your MacBook, we are going to generate a public and a private RSA key on the mac, which will then be use to securely authenticate with the server. This is more secure than just using a password, although this is a very long story.

So first exit the terminal session on your server by typing "exit" in the Terminal.

This should get you back to the OS X command prompt. Now type the below:

```bash
ssh-keygen -t rsa -b 4096
```
Accept all the defaults (just press enter for each and every question).

Now that we have our secure keys we will send the public one on the server.

```bash
ssh-copy-id  <YourUser>@<YourIPAddress>
```
So this should look like:

```bash
ssh-copy-id  pi@192.168.1.10
```

You are now asked for your server password one last time. The public key is then copied on the server, which will recognize your MacBook next time you connect.

Now if you try to connect using :

```bash
ssh pi@192.168.1.10
```
gone is the password prompt!

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

#Choosing the right IDE and extensions

A very nice editor that plays well with Yaml is [Atom](https://atom.io). It's free and works on OsX, Windows and Linux.

Some of the useful extensions that will save you some time are :

Pigment
Yaml lint
Color picker

#Advanced SSH

As you may soon expose your server to the Internet, there are few things you want to put in place so that your ssh access from outside your home is secure. And even though SSH stands for Secure Shell there remains few things you need to do.
