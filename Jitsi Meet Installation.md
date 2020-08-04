


# Install lua and dependencies

Enter in sudo user

```bash
sudo su
```

Run the follow script to install lua and her dependencies, and prosody with fixes.
After finish, VM will be restarted
```bash
cd &&
apt-get update -y &&
apt-get install gcc -y &&
apt-get install unzip -y &&
apt-get install lua5.2 -y &&
apt-get install liblua5.2 -y &&
apt-get install luarocks -y &&
luarocks install basexx &&
apt-get install libssl1.0-dev -y &&
luarocks install luacrypto &&
mkdir src &&
cd src &&
luarocks download lua-cjson &&
luarocks unpack lua-cjson-2.1.0.6-1.src.rock &&
cd lua-cjson-2.1.0.6-1/lua-cjson &&
sed -i 's/lua_objlen/lua_rawlen/g' lua_cjson.c &&
sed -i 's|$(PREFIX)/include|/usr/include/lua5.2|g' Makefile &&
luarocks make &&
luarocks install luajwtjitsi &&
cd &&
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add - &&
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list &&
apt-get update -y &&
apt-get upgrade -y &&
apt-get install prosody -y &&
chown root:prosody /etc/prosody/certs/localhost.key &&
chmod 644 /etc/prosody/certs/localhost.key &&
sleep 2 &&
shutdown -r now
```

# Install jitsi-meet

Enter in sudo user
```bash
sudo su
```

After reboot, run the folow script
```bash
cd &&
cp /etc/prosody/certs/localhost.key /etc/ssl &&
apt-get install nginx -y &&
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add - &&
sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list" &&
apt-get -y update &&
apt-get install jitsi-meet -y &&
apt-get install jitsi-meet-tokens -y
```

# Generate a Let's Encrypt certificate (optional, recommended)
In order to have encrypted communications, you need a TLS certificate. The easiest way is to use Let's Encrypt.

Note: Jitsi Meet mobile apps require a valid certificate signed by a trusted Certificate Authority (such as a Let's Encrypt certificate) and will not be able to connect to your server if you choose a self-signed certificate.

Simply run the following in your shell:

```bash
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

Note that this script uses the HTTP-01 challenge type and thus your instance needs to be accessible from the public internet. If you want to use a different challenge type, don't use this script and instead choose I want to use my own certificate during jitsi-meet installation.

# Open necessary ports

Enable ufw 
```bash
ufw enable
```

Open ports
```bash
ufw allow in 22/tcp &&
ufw allow in openssh &&
ufw allow in 80/tcp &&
ufw allow in 443/tcp &&
ufw allow in 4443/tcp &&
ufw allow in 5222/tcp &&
ufw allow in 5347/tcp &&
ufw allow in 10000:20000/udp
```

# Config prosody

Open `/etc/prosody/prosody.cfg.lua` and

Add above lines after admins object
```
admins = {}

component_ports = { 5347 }
component_interface = "0.0.0.0"
```
 
 change 
```
c2s_require_encryption=true
```
to
```
c2s_require_encryption=false
```

and check if on end of file has
```
Include "conf.d/*.cfg.lua"
```

# Prosody manual plugin configuration

### Setup issuers and audiences 

Open `/etc/prosody/conf.avail/<host>.cfg.lua`

and add above lines with your issuers and audiences

```
asap_accepted_issuers = { "jitsi", "smash" }
asap_accepted_audiences = { "jitsi", "smash" }
```

### Under you domain config change authentication to "token" and provide application ID, secret and optionally token lifetime:

```
VirtualHost "jitmeet.example.com"
    authentication = "token";
    app_id = "example_app_id";             -- application identifier
    app_secret = "example_app_secret";     -- application secret known only to your token
```

### To access the data in lib-jitsi-meet you have to enable the prosody module mod_presence_identity in your config.

```
VirtualHost "jitmeet.example.com"
    modules_enabled = { "presence_identity" }
```

### Enable room name token verification plugin in your MUC component config section:

```
Component "conference.jitmeet.example.com" "muc"
    modules_enabled = { "token_verification" }
```

### Setup guest domain
```
VirtualHost "guest.jitmeet.example.com"
    authentication = "token";
    app_id = "example_app_id";
    app_secret = "example_app_secret";
    c2s_require_encryption = true;
    allow_empty_token = true;
```

### Enable guest domain in config.js
Open your meet config in `/etc/jitsi/meet/<host>-config.js` and enable
```js
var config = {
    hosts: {
        ... 
        // When using authentication, domain for guest users.
        anonymousdomain: 'guest.jitmeet.example.com',
        ...
    },
    ...
    enableUserRolesBasedOnToken: true,
    ...
}
```

### Edit jicofo sip-communicator in `/etc/jitsi/jicofo/sip-communicator.properties`
```
org.jitsi.jicofo.auth.URL=XMPP:jitmeet.example.com
org.jitsi.jicofo.auth.DISABLE_AUTOLOGIN=true
```

### Edit jicofo config in `/etc/jitsi/jicofo/config`
SET the follow configs
```
JICOFO_HOST=jitmeet.example.com
```

### And edit videobridge config in `/etc/jitsi/videobridge/config`

Replace 
```
JVB_HOST=
```
TO
```
JVB_HOST=jitmeet.example.com
```

And add after `JAVA_SYS_PROPS`
```
JAVA_SYS_PROPS=...
AUTHBIND=yes
```

Then, restart all services
```bash
systemctl restart nginx prosody jicofo jitsi-videobridge2
```


# Helpers

Restart all services
```bash
systemctl restart prosody jicofo jitsi-videobridge2 jigasi
```

Prosody logs
```bash
tail -f -n 350 /var/log/prosody/prosody.log
```

Jicofo logs
```bash
tail -f -n 350 /var/log/jitsi/jicofo.log
```

Jigasi logs
```bash
tail -f -n 350 /var/log/jitsi/jigasi.log
```

JVB logs
```bash
tail -f -n 350 /var/log/jitsi/jvb.log
```

Register Jibri
```
prosodyctl register <USER> auth.<DOMAIN> <PASSWORD>
```



# Manual react-jitsi-meet

## Install NodeJS

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo bash - &&
apt-get install nodejs -y
```

Clone your repository,

after clone run 
```bash
sudo npm i &&
make
```

After installation, setup Nginx in `/etc/nginx/sites-available/<host>.conf`

```bash
server {
    ...
    location ^~ /.well-known/acme-challenge/ {
        root         /app/jitsi-meet;
    }
    ....
}

server {
    ...
    root /app/jitsi-meet;
    ...
    location ~ ^/(libs|css|static|images|fonts|lang|sounds|connection_optimization|.well-known)/(.*)$
    {
        add_header 'Access-Control-Allow-Origin' '*';
        alias /app/jitsi-meet/$1/$2;
    }
    ...
}
```
