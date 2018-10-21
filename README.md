#Enable SSH:
[Raspberry docs to enable ssh](https://www.raspberrypi.org/documentation/remote-access/ssh/)

#Change the default password!
just use the `passwd` command

#Enable the camera and force sound through 3.5mm jack
`sudo raspi-config`
- Navigate to interface->enable camera
- Navigate to advanced->sound->force 3.5mm

# Install some basic packages
`sudo apt-get update; sudo apt-get install vim git cmake`

# Connect to wifi by default
Add the following to `/etc/network/interfaces`, update your ssid and psk appropriately

    auto lo
     
    iface lo inet loopback
    iface eth0 inet dhcp
     
    allow-hotplug wlan0
    auto wlan0
     
     
    iface wlan0 inet dhcp
        wpa-ssid "JetFuelCantMeltDankMemes"
        wpa-psk "password"

# Install uv4l
This is the program that actually receives and serves the video streams
Do these as root
`curl http://www.linux-projects.org/listing/uv4l_repo/lpkey.asc | apt-key add -`
`echo deb http://www.linux-projects.org/listing/uv4l_repo/raspbian/stretch stretch main >> /etc/apt/sources.list`
`apt-get update`
`apt-get install uv4l uv4l-raspicam uv4l-raspicam-extras uv4l-server uv4l-uvc uv4l-xscreen uv4l-mjpegstream uv4l-dummy uv4l-raspidisp uv4l-webrtc uv4l-renderer`

# Clone the repo (If you haven't already)
`git clone <url to Chedd_Conference_Center repo>`

In `/etc/uv4l/uv4l-raspicam.conf`
Copy this to the bottom of the file (Modify your url names for private key path if necessary):

    server-option = --enable-www-server=yes
    server-option = --www-port=443
    server-option = --enable-webrtc-video=yes
    server-option = --enable-webrtc-audio=no
    server-option = --webrtc-receive-datachannels=yes
    server-option = --webrtc-receive-audio=yes
    server-option = --www-use-ssl=yes
    server-option = --www-ssl-private-key-file=/etc/letsencrypt/live/www.ssmackey.org/privkey.pem
    server-option = --www-ssl-certificate-file=/etc/letsencrypt/live/www.ssmackey.org/fullchain.pem
    server-option = --www-root-path=/home/pi/ccs/
    server-option = --www-webrtc-signaling-path=/webrtc
    server-option = --webrtc-renderer-fullscreen=yes


And set the following settings to yes, if they are no or not present
`nopreview = yes`
`fullscreen = yes`


#Setup letsencrypt with certbot
[certbot documentation](https://certbot.eff.org/lets-encrypt/debianstretch-other)

# Certbot quick instructions
`sudo apt-get install certbot`
`sudo certbot certonly --standalone -d www.ssmackey.org`
Per the instructions this does the following:
This command will obtain a single cert for example.com, www.example.com, thing.is, and m.thing.is; it will place files below /var/www/example to prove control of the first two domains, and under /var/www/thing for the second pair.

# Install screen driver
Note, the `DSPI_BUS_CLOCK_DIVISOR` arg specifies the clockrate of the screen, this must be an even integer. 
30 seems to work, but lower and you should get better performance (refresh rate)

In the <fbcp submodule> directory:
`cmake -DSPI_BUS_CLOCK_DIVISOR=30 -DWAVESHARE35B_ILI9486=ON . `
`make`

Now set the display to start with the system, and make the display sleep when not in use
Copy the binary you just built (fbcp-ili9341) to /usr/bin
Copy chedd_conference_center.service to /etc/systemd/system/
execute as root:
    systemctl daemon-reload
    systemctl enable chedd_conference_center.service
    systemctl start chedd_conference_center.service
With any luck, the screen should turn on...

Add contents of `set_sleep_time.bash to ~/.profile `

# Setup readonly fs (optional)
Follow instructions at [Overlayroot](https://github.com/chesty/overlayroot), contained as a submodule here
its best to do this last
run the script `dorootwork` to change the file system
The `dorootwork` script only allows editing for the current shell...

# Notes and gotchas
The letsencrypt certificate expires every 3 months, will need to repeat the process if it does. Though you can renew before hand
