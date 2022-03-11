# OpenVPN on LAN and WAN
两种模式：虚拟路由模式、桥接模式

但是要注意UDP模式下在网络复杂的环境下会丢包较多。效率比较低下。

多种内网设备目标平台：Linux, Windows7, Windows10, Windows XP, Raspberry Pi (ARM), MIPSEL

----
> 关于两种模式的区别及优劣简要对比。
> <https://security.stackexchange.com/questions/46442/openvpn-tap-vs-tun-mode#:~:text=1%20Answer&text=TAP%20is%20basically%20at%20Ethernet,bridging%20whereas%20TUN%20is%20routing.>


## 虚拟路由模式 TUN：
### Linux
<https://github.com/Nyr/openvpn-install>
sh脚本，直接运行。
### Windows7
官网下载openvpn-install，NAS下载链接：
在安装过程中勾选 easyrsa选项。  
**Prerequisites:**
- A machine dedicated to running the VPN (This can be a server hosted somewhere or just a PC in your lounge)
- Static IP for the server (I used 192.168.0.3 assigned by my router)
- Your chosen port forwarded for the VPN to work (I have a garbage Virgin router, but it still lets me port forward. I used port 443 since it's not usually blocked on things like corporate networks)
- Static external IP or dynamic DNS (I use ChangeIP for mine since it's free)
Installing OpenVPN Server
- Download from the official site (You will want the Windows installer)
- Install OpenVPN and make sure to check the "EasyRSA" box
- Click "Install" when prompted to install the TAP driver
Creating The CA Certificate
Parts of this next bit are from Bobby Allen's guide
- Open CMD as admin and paste these commands in
```bash 
cd "C:\Program Files\OpenVPN\easy-rsa"
init-config.bat
notepad C:\Program Files\OpenVPN\easy-rsa\vars.bat
```
- When notepad opens, change the following:
  >   set DH_KEY_SIZE=2048 --> set DH_KEY_SIZE=1024
  >   set KEY_SIZE=4096 --> set KEY_SIZE=1024
- You can also change the certificate fields. These don't matter too much, so you can put whatever you want in them. I chose to put my DDNS in mine for example:
  >   set KEY_COUNTRY=US --> GB
    >
  >   set KEY_PROVINCE=CA --> GB
    >
  >   set KEY_CITY=SanFrancisco --> GB
    >
  >   set KEY_ORG=OpenVPN --> sample.ddns.us
    >   
  >   set KEY_EMAIL=mail@host.domain --> sample.ddns.us@mail.com
    >
  >   set KEY_CN=changeme --> sample.ddns.us
    >
  >   set KEY_NAME=changeme --> sample.ddns.us
    >
  >   set KEY_OU=changeme --> sample.ddns.us
    >
  >   set PKCS11_MODULE_PATH=changeme --> sample.ddns.us
    >
  >   set PKCS11_PIN=1234 (keep as the same)

- Save and close the notepad file
- Next, run these commands:
```bash
cd "C:\Program Files\OpenVPN\easy-rsa"
vars.bat
clean-all.bat
build-ca.bat
```
- Press enter through "build-ca.bat" apart from these fields. Enter "ca" without the quotation marks for these
  >   Common name: ca
  >   Name: ca  
- Building The Server Certificate
- Next we want to build the server key. Run the below command
```bash 
build-key-server.bat server 
```
- Like above, set the "common name" and "name" for this as "server"
  >   Common name: server
  >   Name: server
- Press enter through the rest and enter "y" for signing and committing the cert
Building Client Certificate(s)
- Next we need to make a certificate for each client. In my case my only client is my android phone, so I just made the one and called it "Client". You can replace this with a friendly name if you want
```bash
vars.bat
build-key.bat your-device-name-here
```
- Again, set the "common name" and "name" for this as your client's name
>   Common name: your-device-name-here
> 
>   Name: your-device-name-here
- Press enter through the rest and enter "y" for signing and committing the cert
Building DH Parameters
- In the CMD window, run
```bash
   build-dh.bat
```
- This will generate "dh1024.pem" in the config folder
Building TLS Key
- Run these commands to generate ta.key (More info about this can be found here)
```bash
cd "C:\Program Files\OpenVPN\bin"
openvpn --genkey --secret ta.key
move "ta.key" "C:\Program Files\OpenVPN\config"
```
- Moving Server Files To Config Folder
- Run the below commands to move the files you generated for the server to the config folder for it to run
```bash
cd "C:\Program Files\OpenVPN\easy-rsa\keys"
move "ca.crt" "C:\Program Files\OpenVPN\config"
move "dh1024.pem" "C:\Program Files\OpenVPN\config"
move "server.crt" "C:\Program Files\OpenVPN\config"
move "server.key" "C:\Program Files\OpenVPN\config"
```
- Creating .OVPN Files
- Run these commands to prep your client and server .ovpn files
```bash
md "C:\Program Files\OpenVPN\clients"
cd "C:\Program Files\OpenVPN\sample-config"
copy "server.ovpn" "C:\Program Files\OpenVPN\config"
copy "client.ovpn" "C:\Program Files\OpenVPN\clients"
```
Modifying Server.ovpn
- Before going forward I'd strongly recommend installing Notepad++, purely because it makes all the next steps much easier
- Run this command to open "server.ovpn" in notepad++
```bash
"C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\config\server.ovpn"
```
- Replace all the text in "server.ovpn" with this config file
```
port 1194
proto udp
dev tun
ca "C:\\Program Files\\OpenVPN\\config\\ca.crt"
cert "C:\\Program Files\\OpenVPN\\config\\server.crt"
key "C:\\Program Files\\OpenVPN\\config\\server.key"
dh "C:\\Program Files\\OpenVPN\\config\\dh1024.pem"
tls-auth "C:\\Program Files\\OpenVPN\\config\\ta.key" 0
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
cipher AES-256-CBC
comp-lzo
persist-key
persist-tun
status openvpn-status.log
verb 3
explicit-exit-notify 1
```

- If you want to run OpenVPN on a different port, make sure to forward that port on your router and modify it in the "server.ovpn" file
Modifying your-device-name-here.ovpn
- Run these commands to open up all of your certs and keys in Notepad++
-   "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\clients\client.ovpn"
-   "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\config\ca.crt"
-   "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\easy-rsa\keys\client.crt"
-   "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\easy-rsa\keys\client.key"
-   "C:\Program Files (x86)\Notepad++\notepad++.exe" "C:\Program Files\OpenVPN\config\ta.key"
- Replace all the text in the "your-device-name-here.ovpn" with the text from this config
```OpenVPN Config
client
dev tun
proto udp
remote example.ddns.us 443
resolv-retry infinite
nobind
comp-lzo
persist-key
persist-tun
remote-cert-tls server
key-direction 1
cipher AES-256-CBC
verb 3
<ca>
-----BEGIN CERTIFICATE-----
ca.crt here
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
your-device-name-here.crt here
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
your-device-name-here.key here
-----END PRIVATE KEY-----
</key>
<tls-auth>
-----BEGIN OpenVPN Static key V1-----
ta.key here
-----END OpenVPN Static key V1-----
</tls-auth>
```
- Replace "example.ddns.us 443" with whatever your DDNS address or static IP is and the port you're using
- Now go through each crt and key file, replacing "X here" with the keys (This is so that the data from the files is in the 1 .ovpn file. This is better for cross device compatibility)
- Save "your-device-name-here.ovpn"
- You can now copy "your-device-name-here.ovpn" to whatever device you're using. I copied mine to my Android phone using Google Drive (If you're using Android, install the OpenVPN Connect app)
Configuring The Server To Allow Traffic
- This next part makes it so that your VPN sends all traffic through it
- Open "regedit" and paste this text into the top address bar
  ```Regedit
  Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
  ```
- Double click "IPEnabledRouter" and enter the value data as "1"
- Now open "run" and enter "services.msc"
- Find "Routing and Remote Access", right click it, go into properties and change the "Startup type" to "Automatic"
- Now find "OpenVPNService", right click it, go into properties and change the "Startup type" to "Automatic"
- Now go into Control Panel and navigate to "Network and Sharing Center" then click "Change Adapter Settings" on the left
- Find the adapter that has "TAP" underneath it, then rename that adapter to "TAP"
- Right click on your adapter that has an internet connection and go to "properties"
- Select "Sharing" at the top and check the box for "Allow other network users to connect..."
- You may have a drop down box to select what networks can run through it (this usually shows up if you've got Hyper-V or something like that running on it). If that shows up, click the drop down box and select "TAP"
- If you only have the wired Ethernet and "TAP" showing up, there may be no drop down box. I haven't been able to extensively test this, but you should just be able to check the box without a dropdown
Testing
- From here, that should be it. You may need to reboot your server for it to work
- I tested mine by switching my phone to 4G and connecting to the VPN using the "Client.OVPN" I generated
- I only got this working yesterday, so I can confirm it works on my 4G but not on other wireless networks
- You can find out if it works easily by Googling your IP, then connecting the VPN and Googling it again
Please drop a comment below if you'd like any help

### Windows10

### Raspberry Pi (ARM)

### PandoraBox (MIPSEL)
 - 可以参考<https://www.itdaan.com/tw/e82d330893902223d7e8256474237758>
 - 
------
## 桥接模式 TAP：

### Linux

### Windows7

### Windows10

### Raspberry Pi (ARM)

### PandoraBox (MIPSEL)

