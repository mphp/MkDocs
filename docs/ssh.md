#Setting up ssh
Now that your Raspberry or PC is installed with Ubuntu, Raspbian or any other Linux flavor, it is time to access it. As it will be used as a server, there is no need to plug a monitor, or to have it around you. It can lie in a closet, next to your router. As long as it is network wired.

SSH is the easiest and most secure way to access your server. You will need access to manage services running there, as well as installing new software depending on your usage. It's also needed so that you can change the default password of the default user, specifically on the Raspberry Pi.

!!! warning " "
    Even though SSH stands for Secure Shell you need additional steps to make it robust and start using it from outside of your home network.

    Those steps are described [here](#advanced-ssh).

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

##Advanced SSH

As you may soon expose your server to the Internet, there are few things you want to put in place so that your ssh access from outside your home is secure.

###SSH Daemon configuration

###fail2ban
