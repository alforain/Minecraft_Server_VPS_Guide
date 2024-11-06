# Minecraft Server VPS Guide
### Prerequisites
Have a domain and a vps running an up-to-date version of Debian or one of its derivatives with ssh installed and setup on both your machine and vps. For this guide I use [contabo](https://contabo.com/) for the vps and [namecheap](https://www.namecheap.com/) for the domain.

### Install OpenJdk 21 and it's dependencies
```
# apt install software-properties-common openjdk-21-jdk
```
### Create a new user
Create a new user account, this is for security reasons:
```
adduser minecraft
```
Set permissions for the user so that it can use the sudo command:
```
usermod -aG sudo minecraft
```
### Download and install the server itself
Switch to the mincraft user account:
```
su - minecraft
```
Create a directory for the server and use the `cd` command to go into that directory:
```
mkdir minecraft_server
```
```
cd minecraft_server
```
Download the server by checking the official minecraft server download, right click the .jar file, copy it and insert it into the `wget` command:
```
wget [your link here]
```
### Start the server and agree to the eula
Start the server to generate the required files,  
(Use the `Ctrl+Z` shortcut to kill the server once the required files have been generated):
```
java -Xmx4096M -Xms4096M -jar server.jar nogui 
```
Edit the eula to agree it it,  
(Change `eula=false` to `eula=true` and save the file):
```
nano eula.txt
```
```
eula=true
```
Restart the server and exit with `Ctrl+Z`:
```
java -Xmx4096M -Xms4096M -jar server.jar nogui
```
### Configure server properties [Optional]
Edit the `server.properties` file, set default user permisions and enforce a whitelist:
```
nano server.properties
```
```
enforce-whitelist=true
```
```
op-permission-level=0
```
Edit the `whitelist.json` file and add whitelisted users,  
(To get the uuid of a player go to [mcuuid.net](https://mcuuid.net), copy the full uuid and paste it into the file):
```
nano whitelist.json
```
```
[
  {
    "uuid": "[insert uuid here]",
    "name": "[insert username here]"
  },
  {
    "uuid": "[insert uuid here]",
    "name": "[insert username here]"
  }
]
```
Edit the `ops.json` file and add mods and admins,
(Information on user levels are avalible [here](https://minecraft.wiki/w/Permission_level):
```
nano ops.json
```
```
[
   {
      "uuid":"[insert uuid here]",
      "name":"[insert username here]",
      "level":4
   },
   {
      "uuid":"[insert uuid here]",
      "name":"[insert username here]",
      "level":4
   }
]​
```
### Create systemd link
Go back to the root user:
```
su -
```
Create a `systemd` service for minecraft:
```
nano /etc/systemd/system/minecraft.service
```
```
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=minecraft
WorkingDirectory=/home/minecraft/minecraft_server

# Remove the session.lock file before starting the server
ExecStartPre=/bin/rm -f /home/minecraft/minecraft_server/world/session.lock
ExecStart=/usr/bin/java -Xmx4096M -Xms4096M -jar /home/minecraft/minecraft_server/server.jar nogui
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Reload `systemd` to recognize the new service file:
```
systemctl daemon-reload
```
Start the Minecraft server with `systemd`:
```
systemctl start minecraft
```
Enable the service to start on boot:
```
systemctl enable minecraft
```
### Link the server to a domain name
Allow traffic on port 25565 using the `ufw` command:
```
ufw allow 25565
```
Find the ip address of the server:
```
ip a
```
Create an A record and SRV record by going to the DNS settings of your domain registrar:
| **A Record** |             |         |
|--------------|-------------|---------|
| **Host**     | **Value**   | **TTL** |
| mc           | [insert IP address here] | Automatic |

| **SRV Record** |             |            |          |        |                    |
|----------------|-------------|------------|----------|--------|--------------------|
| **Service**    | **Protocol**| **Priority** | **Weight** | **Port** | **Target** |
| _minecraft     | _tcp.mc     | 0          | 5        | 25565  | mc.[yourdomain]    |

### Closing notes
You're now done, and it should be up and running! If you found this tutorial useful please star this reposirory :star:
