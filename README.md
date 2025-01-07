# XLXD Server Setup XLX123

## Credentials

- USER: **xlxserver**
- STATIC IP: **192.168.1.15**
- GATEWAY IP: **192.168.1.1**
- DNS SERVER: **192.168.1.1**
- XLX REFLECTOR: **XLX123** (Example)


## Opening and forwarding ports on router:

- UDP port 8880 (DMR+ DMO mode)
- UDP port 10002 (XLX interlink)
- UDP port 10100 (AMBE controller port)
- UDP port 10101 - 10199 (AMBE transcoding port) # only needed if your transcoder and reflector aren't running on the same domain.
- UDP port 12345 - 12346 (Icom Terminal presence and request port)
- UPD port 20001 (DPlus protocol)
- UDP port 30001 (DExtra protocol)
- UDP port 30051 (DCS protocol)
- UDP port 40000 (Icom Terminal dv port)
- UDP port 42000 (YSF protocol)
- UDP port 62030 (MMDVM protocol)

# [ RaspberryPi installation ]
Installation of XLXD server on raspberryPi 64bit no desktop

Create the following user when flashing the SD card:
```
xlxserver
```
## Distribution Configure / Update / Upgrade
```
sudo raspi-config
```
**Expand file system**
> Advanced Options > Expand Filesystem > Finish and reboot

**add user to to sudoers & dialout group:**
```
sudo usermod -aG sudo xlxserver
sudo usermod -aG dialout xlxserver
sudo reboot
```

**Update and possibly upgrade packages**
```
sudo apt-get update && sudo apt-get upgrade -y
``` 

# [ Debian 12 installation ]

**make xlxserver member of sudoers & dialout group**
```
su -
apt-get install sudo
usermod -aG sudo xlxserver
usermod -aG dialout xlxserver
reboot now
```

**Update and possibly upgrade packages**
```
sudo apt-get update && sudo apt-get upgrade -y
```



# [ Raspberry PI - Create Static IP ]
**If Network Manager is not already installed**
```
sudo apt install network-manager
```
See which name corresponds to the Ethernet connection
```
sudo nmcli con show
```
This may be Ethernet connection **"Wired connection 1"**.

**Now set a static IP (192.168.1.15) for "Wired connection 1"**
```
sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.15/24
sudo nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.1
sudo nmcli con mod "Wired connection 1" ipv4.dns "192.168.1.1"
sudo nmcli con mod "Wired connection 1" ipv4.method manual
sudo nmcli con up "Wired connection 1"
```



# [ Debian 12 - Create Static IP ]

**Open network config and adjust**
```
sudo nano /etc/network/interfaces
```
**Change:**
```
allow-hotplug enp1s0
iface enp1s0 inet dhcp
```
**In to:**
```
auto enp1s0
iface enp1s0 inet static
    address 192.168.1.15
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.1
```
**Restart Network**
```
sudo systemctl restart networking
```


### PLEASE NOTE!, a possible SSH connection will now be lost and must be reconnected with the newly set IP address.


# [ Install GIT, Apache, PHP and various tools ]
```
sudo apt-get install git git-core apache2 php libapache2-mod-php php-cli php-xml php-mbstring php-curl build-essential -y
```

**Set PHP Timezone correctly, pay attention to the php version**
```
sudo sed -i 's|;date.timezone =|date.timezone = "Europe/Amsterdam"|' /etc/php/8.2/apache2/php.ini
sudo systemctl restart apache2
```


# [ Install XLXD Server ]

**Install latest version XLXD Server from GitHub**
```
cd ~
git clone https://github.com/LX3JL/xlxd.git
```

**Next step:**
**Configure xlxd service with your XLX server number and XLX server host ip address**

***example:***

XLX Server Number: **123**

XLX Server IP: **192.168.1.15**

```
cd ~/xlxd/src/
make clean
sudo make
sudo make install
sudo cp ~/xlxd/scripts/xlxd /etc/init.d/xlxd
sudo sed -i 's|ARGUMENTS="XLX999 192.168.1.240 127.0.0.1"|ARGUMENTS="XLX123 192.168.1.15 127.0.0.1"|' /etc/init.d/xlxd
cd ~
```

## Declare the service for automatic startup and shutdown
```
sudo update-rc.d xlxd defaults
```

## !!!! IMPORTANT !!!!
**When you create a new reflector /xlxd/callinghome.php will be created only ONCE! and UNRETRIEVABLE!, BACK IT UP!!!!**

**This hash file is linked to your XLX number**


## Copy Dashboard & restore own configuration of dashboard to /var/www/dashboard/
```
sudo cp -r ~/xlxd/dashboard /var/www
sudo chmod 755 -R /var/www/dashboard/
sudo sed -i 's|DocumentRoot /var/www/html|DocumentRoot /var/www/dashboard|' /etc/apache2/sites-available/000-default.conf
sudo systemctl restart apache2
sudo service xlxd start
```

## Configure XLXD Dashboard ( /var/www/dashboard/pgs/config.inc.php )
```
sudo nano /var/www/dashboard/pgs/config.inc.php
```

**Contents of **config.inc.php**, adjust accordingly**

***To show your XLX Server in the Reflector listing***
-  $CallingHome['Active'] = true;

```
<?php
/*
Possible values for IPModus

HideIP
ShowFullIP
ShowLast1ByteOfIP
ShowLast2ByteOfIP
ShowLast3ByteOfIP

*/

$Service     = array();
$CallingHome = array();
$PageOptions = array();
$VNStat      = array();

$PageOptions['ContactEmail']                         = 'your_email';	// Support E-Mail address

$PageOptions['DashboardVersion']                     = '2.4.2';		// Dashboard Version

$PageOptions['PageRefreshActive']                    = true;		// Activate automatic refresh
$PageOptions['PageRefreshDelay']                     = '10000';		// Page refresh time in miliseconds

$PageOptions['NumberOfModules']                      = 10;		// Number of Modules enabled on reflector

$PageOptions['RepeatersPage'] = array();
$PageOptions['RepeatersPage']['LimitTo']             = 99;		// Number of Repeaters to show
$PageOptions['RepeatersPage']['IPModus']             = 'ShowFullIP';	// See possible options above
$PageOptions['RepeatersPage']['MasqueradeCharacter'] = '*';		// Character used for  masquerade

$PageOptions['PeerPage'] = array();
$PageOptions['PeerPage']['LimitTo']                  = 99;		// Number of peers to show
$PageOptions['PeerPage']['IPModus']                  = 'ShowFullIP';	// See possible options above
$PageOptions['PeerPage']['MasqueradeCharacter']      = '*';		// Character used for  masquerade

$PageOptions['LastHeardPage']['LimitTo']             = 39;		// Number of stations to show

$PageOptions['ModuleNames'] = array();					// Module nomination
$PageOptions['ModuleNames']['A']                     = 'Int.';
$PageOptions['ModuleNames']['B']                     = 'Regional';
$PageOptions['ModuleNames']['C']                     = 'National';
$PageOptions['ModuleNames']['D']                     = '';

$PageOptions['MetaDescription']                      = 'XLX is a D-Star Reflector System for Ham Radio Operators.';	// Meta Tag Values, usefull for Search Engine
$PageOptions['MetaKeywords']                         = 'Ham Radio, D-Star, XReflector, XLX, XRF, DCS, REF, ';		// Meta Tag Values, usefull forSearch Engine
$PageOptions['MetaAuthor']                           = 'LX1IQ';								// Meta Tag Values, usefull for Search Engine
$PageOptions['MetaRevisit']                          = 'After 30 Days';							// Meta Tag Values, usefull for Search Engine
$PageOptions['MetaRobots']                           = 'index,follow';							// Meta Tag Values, usefull for Search Engine

$PageOptions['UserPage']['ShowFilter']               = true;								// Show Filter on Users page
$PageOptions['Traffic']['Show']                      = false;								// Enable vnstat traffic statistics
$PageOptions['IRCDDB']['Show']                       = true;        // Show liveircddb, set it to false if you are running your db in https 

$PageOptions['CustomTXT']                            = '';					// custom text in your header   

$Service['PIDFile']                                  = '/var/log/xlxd.pid';
$Service['XMLFile']                                  = '/var/log/xlxd.xml';

$CallingHome['Active']                               = false;					// xlx phone home, true or false
$CallingHome['MyDashBoardURL']                       = 'http://your_dashboard';			// dashboard url
$CallingHome['ServerURL']                            = 'http://xlxapi.rlx.lu/api.php';		// database server, do not change !!!!
$CallingHome['PushDelay']                            = 600;					// push delay in seconds
$CallingHome['Country']                              = "your_country";				// Country
$CallingHome['Comment']                              = "your_comment";				// Comment. Max 100 character
$CallingHome['HashFile']                             = "/tmp/callinghome.php";			// Make sure the apache user has read and write permissions in this folder.
$CallingHome['LastCallHomefile']                     = "/tmp/lastcallhome.php";			// lastcallhome.php can remain in the tmp folder 
$CallingHome['OverrideIPAddress']                    = "";					// Insert your IP address here. Leave blank for autodetection. No need to enter a fake address.
$CallingHome['InterlinkFile']                        = "/xlxd/xlxd.interlink";			// Path to interlink file

$VNStat['Interfaces']                                = array();
$VNStat['Interfaces'][0]['Name']                     = 'eth0';
$VNStat['Interfaces'][0]['Address']                  = 'eth0';
$VNStat['Binary']                                    = '/usr/bin/vnstat';

/*   
include an extra config file for people who dont like to mess with shipped config.ing.php   
this makes updating dashboard from git a little bit easier   
*/   
 
if (file_exists("../config.inc.php")) {   
 include ("../config.inc.php");  
}   

?>
```


# [ Download dmrid.dat from XLXAPI server ]
```
sudo wget -O /xlxd/dmrid.dat http://xlxapi.rlx.lu/api/exportdmr.php
```


# [ Add XLXEcho Daemon PEER (Optional) ]

**Install latest version XLXEcho from GitHub**
```
cd ~
git clone https://github.com/narspt/XLXEcho.git
cd XLXEcho
sudo gcc -o xlxecho xlxecho.c
sudo cp xlxecho /xlxd
cd ~
```

## Add PEER to xlxd.interlink
```
sudo nano /xlxd/xlxd.interlink
```
***Example:***
- Peer name: **ECHO**
- Peer host: **127.0.0.1**
- Peer Module: **I**

Add **ECHO 127.0.0.1 I**
```
ECHO 127.0.0.1 I
```


## Add service for XLXEcho
```
sudo nano /etc/systemd/system/xlxecho.service
```

Copy and Paste and save file
```
[Unit]
Description=XLXEcho Service
After=xlxd.service
Requires=xlxd.service

[Service]
ExecStart=/xlxd/xlxecho ECHO 127.0.0.1
Restart=always
User=root
WorkingDirectory=/xlxd
StandardOutput=append:/xlxd/xlxecho.log
StandardError=append:/xlxd/xlxecho.log

[Install]
WantedBy=multi-user.target
```

**Start XLXEcho Daemon**
```
sudo chmod 755 /etc/systemd/system/xlxecho.service
sudo systemctl daemon-reload
sudo systemctl enable xlxecho.service
sudo systemctl start xlxecho.service
```



# [ DMRBot installation (Optional) ]

If Python is not yet installed, install Python.
Otherwise continue with PIP setup.

```
sudo apt install python3-full
```

**pip setup**
```
sudo apt install -y python3-pip
```

**Install Python Google Text to Speech module**
```
sudo pip install --upgrade --break-system-packages gtts
```

**Install FFMpeg**
```
sudo apt install ffmpeg -y
```

**Install latest version DMRBot ​​from GitHub**
```
cd ~
git clone https://github.com/narspt/DMRBot.git
```

**OpenAI API Key**

You will need an openAI API Key. You can get one here: https://platform.openai.com/signup
It has to be stored in: **~/DMRBot/openai_api_key.txt**

```
sudo nano ~/DMRBot/openai_api_key.txt
```
***Now Copy and Paste and save your API key***


**Adjust configuration and Compile DMRBot**
```
cd ~/DMRBot
sudo sed -i 's|#define AMBE_ENCODE_GAIN -5|#define AMBE_ENCODE_GAIN 5|' /home/xlxserver/DMRBot/xrfbot.c
sudo gcc -o xrfbot xrfbot.c
sudo chmod 666 *.wav *.mp3 *.json
cd ~
```

## Start DMRBot ​​at boot

**If Screen is not yet installed**
```
sudo apt install screen
```

**Adds line to cron**

***Important!, Copy and paste this block in its entirety!***
```
crontab -l > mycron
echo "@reboot screen -dmS dmrbot /home/xlxserver/DMRBot/xrfbot DVBOT XRF123:J:192.168.1.15:30001 192.168.1.15:2460" >> mycron
crontab mycron
rm mycron
```

**Screen options**
```
screen -S dmrbot -dm bash runbot.sh     # Start runbot.sh directly as screen
screen -ls                              # List of active screens
screen -r dmrbot                        # Reconnect Screen: dmrbot
screen -X -S dmrbot quit                # Close screen: dmrbot
```

**Options when in a Screen**
```
CTRL + A, D                             # Close window without closing screen
CTRL + A, K                             # Kills screen
```


## Manual boot options

**Start DMR Bot**
```
./runbot.sh
```

**Start DMRBot ​​in screen mode**
```
./runscreenbot.sh
```

# [ AMBE Daemon Server installation needed for DMRBot (Compatible with CP210x) ]

**Install latest version AMBEserver from github**
```
cd ~
git clone https://github.com/marrold/AMBEServer.git
cd ~/AMBEServer
sudo make
sudo make install
sudo make init-install
sudo sed -i 's|/usr/bin/AMBEserver|/usr/bin/AMBEserver -p 2460 -s 460800|' /etc/init.d/AMBEserver
sudo systemctl daemon-reload
sudo systemctl enable AMBEserver
cd ~
```
