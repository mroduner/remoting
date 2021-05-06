Scripts and Configiration for a User Mode Xpra Application Server
===

Overview
--- 

We're going to configure Xpra as a user service and ~/.xinitrc using KDE Neon 
as our base linux installation.

Really any desktop focused version of Debian should work well enough and 
caveats will be marked as we find them.

Setting up xpra
---

Adding xpra signing key

    $ wget -q https://xpra.org/gpg.asc -O- | sudo apt-key add -

Adding the ppa

    $ wget -q "https://xpra.org/repos/$(grep "DISTRIB_CODENAME" < /etc/lsb-release | sed -r 's/^[^=]+=(.*)$/\1/')/xpra.list" -O xpra.list
    $ sudo mv xpra.list /etc/apt/sources.list.d/

Freshen up apt, and install

    $ sudo apt update && sudo apt install xpra

Moving our helper scripts
---

Not a lot of people use per-user scripts in linux for some reason, so let's
make sure we have a functional ~/bin

    $ mkdir -p ~/bin

Okay, now just the simple matter of copying the script over and setting the 
right permissions

    $ cp start-xpra ~/bin
    $ chmod +x ~/bin/start-xpra

Setting up the systemd unit
---

We're going to move our service unit into user folder

    $ cp xpra-user.service ~/.config/systemd/user/

Next, we need to reload systemd and then enable our service

    $ systemctl --user daemon-reload
    $ systemctl --user enable xpra-user.service

Copy the contents of example.xinitrc to .xinitrc and make it executable

    $ cp example.xinitrc ~/.xinitrc
    $ chmod +x ~/.xinitrc

Go ahead and start the service with systemctl

    $ systemctl --user start xpra-user.service

Connecting to the Xpra server
---

The xpra server is currently configured to advertize via mDNS and is listening 
on port 100000, so we can try connecting to it at 

    http://_hostname_.local:100000

It should direct you to a sign in page, just go ahead and log in with your 
existing account, and you should be greated with a background, simple window
manager, and an xterm.  Good Job! 

If that didn't work, mDNS isn't set up correctly, so let's get
our IP address instead 

    $ ip address  | grep global | sed -r "s/^[^1-9]*([^/]+)[/].*/\1/"

Editing the files
---

So, this is probably not the most useful thing to you, unless you're me, and
prefer to work directly on the command line; Or you're the running with
scissors type and typing that password is just the bridge too far...

To change the application that Xpra will start for you, simply change the 
following line in ~/.xinitrc

    exec konsole

to whatever application you want to run, for instance to run VS code our
command line would look like this:

    exec code -w 

Whatever you do, remember, when the script exits so does your Xpra session, so 
if your program immediatly forks, you'll need to find a way around that (like 
VS Code's '-w' switch)

Now if you exit the Konsole, xpra should shut down, restart, and load the new 
contents of .xinitrc

If you want to change how you authenticate to the xpra server, edit the 
folowing line in ~/bin/start-xpra

    --bind-tcp=0.0.0.0:10000,auth=sys\

The part that says "sys" is the part we can change, for example if you just 
would like to authenticate with a simple password our line would look like
below

    --bind-tcp=0.0.0.0:10000,auth=password:value=somesecret\

Or if you __really__ know what you're doing

    --bind-tcp=0.0.0.0:10000,auth=allow\

I personally recomend using the xpra native client for accessing your remote 
application; using SSH forwarding and authentication really makes things a 
lot more seamless. 