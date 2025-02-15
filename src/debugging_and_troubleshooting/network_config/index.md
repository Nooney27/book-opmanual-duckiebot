(duckiebot-network)=
# Operation - Networking

```{needget}
* A Duckiebot that is initialized according to [](#setup-duckiebot).
* Patience (channel your inner Yoda)
---
* A Duckiebot that you can connect to and that is connected to the internet.
```

The instructions here are ordered in terms of preference, the first being the most preferable and best.

By default on boot your robot will look for a network with a "`duckietown`" SSID, unless you changed it in [the SD card flashing procedure](burn-sd-card). You can connect to your robot wirelessly by connecting to that network.

This page describes how to get your robot connected to the wide-area network (internet).

## Add WiFi Networks without reinitializing the SD card

To add networks at a later stage or modify existing settings, edit the file `wpa_supplicant.conf` in the main partition of the SD card. 

For robots based on Raspberry Pi, (e.g., ` DB17`, ` DB18`, ` DB19`), this file is located at `/etc/wpa_supplicant/wpa_supplicant.conf` in the `root` partition;
For robots based on Nvidia Jetson Nano, (e.g., ` DB21M`), this file is located at `/etc/wpa_supplicant.conf` in the `APP` partition;

New networks can be created by adding a new `network={}` paragraph, and then entering the network information. An example network configuration is shown below:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CH

network={
    id_str="network_1"
    ssid="comnet23243"
    psk="MSNDJWKE32"
    key_mgmt=WPA-PSK
}

network={
    id_str="network_2"
    ssid="TPlink23432"
    psk="ksnbn4wn3"
    key_mgmt=WPA-PSK
}
```

(duckiebot-network-test)=
## Testing if your Duckiebot is Connected to the Internet 

Some networks block pings from passing through, so a better way is to execute on your duckiebot:

    duckiebot $ sudo curl google.com

which will try to download the Google homepage. If it is successful, you should see an output like:

```
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com/">here</A>.
    </BODY></HTML>
```

## Option 1: Connect your Duckiebot to the internet through a WiFi router that you control

If you are working from your home, for example, you simply need to make the Duckiebot connect to your home network. You may have input the proper SSID and password when you initialized the SD card, in which case, your Duckiebot should be connected to the internet already.

If you didn't enter the right SSID and password for your network or you want to change you need to connect to your robot somehow (e.g. with Ethernet) and then edit the file `/etc/wpa_supplicant/wpa_supplicant.conf` as explained in the [Duckiebot initialization procedure](burn-sd-card).

This is the best option.

## Option 2: Bridge the internet connection through your laptop with Ethernet

This method assumes that you can connect your laptop to a network but it is one that you don't control or is not open. For example, on campus many networks are more protected, e.g., with `PEAP`. In that case, it can be difficult to get your configurations right on the Duckiebot. An alternative is bridge the connection between your laptop and your Duckiebot whenever you need internet access on the robot.

### Ubuntu

1. Connect your laptop to a wireless network.
2. Connect the Duckiebot to your laptop via an Ethernet cable.
3. Make a new Ethernet connection:

   1. Network Settings… (or run the command `nm-connection-editor`)
   2. Click "Add"
   3. Type -> Ethernet
   4. Connection Name: "Shared to Duckiebot"
   5. Select "IPV4" tab
   6. Select Method
   7. Select “Shared to other computers”
   8. Click apply.

Now, you should be able to SSH to your Duckiebot:

    $ ssh ![hostname]

```{note}
The next three commands should be executed on your Duckiebot through SSH
```

Check whether you can access the internet from your Duckiebot:

    $ sudo curl google.com

Now, try to pull a Docker image:

    $ sudo docker pull duckietown/rpi-simple-server # This should complete successfully


If the previous command does not work, you may need to change the system date. To do so, run the following command:

    $ sudo date -s "2018-09-18 15:00:00" # Where this is the current date in YYYY-MM-DD HH-mm-ss



### Mac

Untested instructions [here](https://medium.com/@tzhenghao/how-to-ssh-into-your-raspberry-pi-with-a-mac-and-ethernet-cable-636a197d055)

duckiebot-network-push = ## Option 3: Push Docker Images from Laptop 

Since we are primarily using the internet to pull Docker images, we can simply connect the laptop and the Duckiebot then push Docker images from the laptop over SSH like so:


    $ docker save duckietown/![image-name] | ssh -C ![hostname] docker load


Then the image will be available on your Duckiebot.

If you can connect to your laptop (e.g. through a router) but do not have internet access then you can proceed for now, but everytime you see a command starting with:


    $ docker run ...


Note that you will need to pull onto your laptop and push to your Duckiebot in order to load the latest version of the image.

## Troubleshooting

### I cannot ping the Duckiebot

```{trouble}
I cannot ping the Duckiebot (`ping ![robot_name]` does not work).
---
Check if your laptop and Duckiebot are connected to the same network.

Additional debugging steps:

- Step 1: Check that your Raspberry Pi is responsive by observing the blinking LED on the Raspberry Pi.

- Step 2: Connect your Duckiebot with the laptop using the ethernet cable. Check if you are able to ping the Duckiebot. This will provide you an hint if there is an issue with the robot or network.

- Step 3: Check that this file: `/etc/wpa_supplicant/wpa_supplicant.conf` contains all the wifi networks in the correct syntax that you want to connect.

- Step 4: If it's your private access point, then you can access your router, typically connecting to `192.168.0.1`, where you can see all the devices connected. Make sure that both your Duckiebot and your laptop are in the list.

- Step 5: Check the file `~/.ssh/config` has the correct name hostname with `hostname.local` defined.

```


```{trouble}
When I run `ssh ![robot_name].local` I get the error `ssh: Could not resolve hostname ![robot_name].local`.
---
Make sure that your Duckiebot is ON. Connect it to a monitor, a USB mouse and a keyboard. Run the command:

On the duckiebot run: 

``
    $ sudo service avahi-daemon status
``

You should get something like the following:

``
    ● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
       Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled; vendor preset: enabled)
       Active: active (running) since Sun 2017-10-22 00:07:53 CEST; 1 day 3h ago
     Main PID: 699 (avahi-daemon)
       Status: "avahi-daemon 0.6.32-rc starting up."
       CGroup: /system.slice/avahi-daemon.service
               ├─699 avahi-daemon: running [![robot_name_in_avahi].local
               └─727 avahi-daemon: chroot helpe
``

Avahi is the module that in Ubuntu implements the mDNS responder. The mDNS responder is responsible for advertising the hostname of the Duckiebot on the network so that everybody else within the same network can run the command `ping ![robot_name].local` and reach your Duckiebot. Focus on the line containing the hostname published by the `avahi-daemon` on the network (i.e., the line that contains `![robot_name_in_avahi].local`).
If `![robot_name_in_avahi]` matches the `![robot_name]`, go to the next Resolution point.
If `![robot_name_in_avahi]` has the form `![robot_name]-XX`, where `XX` can be any number, modify the file `/etc/avahi/avahi-daemon.conf` as shown below.

Identify the line

``
    use-ipv6=yes
``

and change it to

``
    use-ipv6=no
``

Identify the line

``
    #publish-aaaa-on-ipv4=yes
``

and change it to

``
    publish-aaaa-on-ipv4=no
``

Restart Avahi by running the command

``
   $ sudo service avahi-daemon restart
``
```

```{trouble}
I can SSH to the Duckiebot but not without a password
---
Check the file `~.ssh/config` and make sure you add your `ssh` key there, in case it doesn't exists.

The `init_sd_card` [procedure](setup-duckiebot) should generate a paragraph in the above file in the following format:

``
    Host duckiebot
        User duckie
        Hostname duckiebot.local
        IdentityFile /home/user/.ssh/DT18_key_00
        StrictHostKeyChecking no
``

Do:

``
    $ ssh-keygen -f "/home/user/.ssh/known_hosts" -R hostname.local
``

It will generate a key for you, if it doesn't exists.

```

```{trouble}
Error message appears saying `I cannot communicate with docker`. Also a warning `\"DOCKER_HOST\" is set to ![hostname].local` is present.
---
Unset the `DOCKER_HOST`, running:

``
    $ unset DOCKER_HOST
``    
```


```{trouble}
You can ping the robot, `ssh` into it, start the demos, but the commands 
from the virtual joystick do not seem to reach the robot.
---
A possible cause is that your computer's firewall is blocking the incoming traffic 
from the robot.
Check the settings for the firewall on your computer and make 
sure that any incoming traffic from the IP address of the robot is allowed on all ports. Keep in mind that if your robot's IP address changes, you might need to update the rule.
```
