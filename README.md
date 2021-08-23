# WSL2-WEB-DEV
Example configuration
- Port: `8080`
- Host: `mysite.test`
- IPv4: `192.168.1.100`

## Installation
- Make sure you are running the latest Windows Insider build
- Apache2 installed in WSL2

## Configuration

Firstly you should decide on what you'd like your site's hostname to be. In this example, it is `mysite.test` running on port `8080`
Note your local IPv4 address. In this example, it is `192.168.1.100`

### WSL2 (Ubuntu 20.04)

#### Apache2
- To start Apache2, use the command `sudo service apache2 start`
- Create a VirtualHost file in `/etc/apache2/sites-available`
- In this example, it would be `/etc/apache2/sites-available/mysite.test.conf`
- In this file, paste the following contents, then save
```
Listen 8080

<VirtualHost *:8080>
    ServerAdmin admin@example.com
    ServerName mysite.test
    ServerAlias mysite.test
    DocumentRoot /home/you/laravel/project/public

    <Directory /home/you/laravel/project/public>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
            Require all granted
    </Directory>

    LogLevel debug
    ErrorLog ${APACHE_LOG_DIR}/mysite.test-error.log
    CustomLog ${APACHE_LOG_DIR}/mysite.test-access.log combined
</VirtualHost>
```
- *Note: If using Laravel, the DocumentRoot should be set the the `public` directory in your project. Use the absolute path.
- Run the command: ``a2ensite mysite.test``
- Finally, run the command: ``sudo service apache2 restart``

### /etc/hosts
- Edit `/etc/hosts`, add the following entry to the end, and save
```
192.168.1.100 mysite.test
```
- *Note: This change should take effect without restarting WSL2

### Windows
#### .wslconfig
- Create a file named `.wslconfig` in your user directory (`C:\Users\You`)
- Paste the following contents `.wslconfig` and save
```
[wsl2]
localhostForwarding=true
```

#### etc\hosts
- Edit `c:\windows\system32\drivers\etc\hosts`, add the following entry to the end, and save
```
192.168.1.100 mysite.test
```
- *Note: This change should take effect without restarting Windows

#### Opening Ports
Personally, I put my WSL scripts in my User directory under `.wsl` (`C:\Users\You\.wsl`)
- Paste the following inside of a new file as `C:\Users\You\.wsl\openports.ps1`
```ps1
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
  echo $remoteport;
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

#[Ports]

#All the ports you want to forward separated by coma
$ports=@(8080);

#[Static ip]
#You can change the addr to your ip config to listen to a specific address, but I'd keep it at 0.0.0.0
$addr='0.0.0.0';
$ports_a = $ports -join ",";


#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
  echo "Opened port $port";
}

echo "Success";
```

### Conclusion
- After you've completed these steps, run the command `wsl --shutdown` and/or `Restart-Service LxssManager` (older versions)
- Start wsl with the command `wsl`
- In an elevated PowerShell window, run the command: `Set-ExecutionPolicy unrestricted`
- Then run: `& C:\Users\You\.wsl\openports.ps1`
- Finally, set the ExecutionPolicy back to normal: `Set-ExecutionPolicy default` (or don't)

Any time you restart WSL you must run `openports.ps1`. This can be automated.
Any time you add/remove/edit a VirtualHost in Apache2, you must restart Apache2.
