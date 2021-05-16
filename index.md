# In this project we will Create .NET Core Api Service and run it over Apache2 on Ubuntu 18
## Prerequisites
An Ubuntu 16/18/2.0 /machine
Sudo user (Root user)
Putty to make the life easy

## Install Apache2 server
In your Ubuntu Machine
Go root and install apache server
<pre><code>
$ sudo su
$ sudo apt-get install apache2 -y
$ sudo apt-get install lamp-server^
$ sudo systemctl enable apache2
</code></pre>

Now server is ready, to test your work, open a web browser and paste your machine Ip address
192.168.*.*
<<<Its work>>>

## Create a .Net Core project
Now we will create a web api .net core application using visual studio 2017
In your windows machine
* Open your visual studio
* Follow the wizard to create your web api service

I'll assume that your project name is "ApiService"
Now we will publish it, from the project navigation, right click on the root folder "ApiService" and click publish

We need to create a publish profile
From the side bar, choose "IIS,FTP,etc" then click on Create Profile
In the Connection change "Wep Deploy" to "File system" and choose your fav destination, in my case i choose "~/Desktop/Publish"
In Settings do all the following:
* Keep release as is
* Target framework is the version of your net core "In my case it was 3 or 2.2"
* Deployment Mode is Framework-Deployment
* Target Runtime is Linux-x64

Click "save" then publish, you will find the publish folder in the Desktop, keep it for now.

## .Net Configuration in Ubuntu machine
We need to configure .Net core in the ubuntu server (Go root)

In your Ubuntu machine
Lets install .Net Core 2.2 (you can install any latest framework toy want)
<pre><code>
$ wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
$ sudo dpkg -i packages-microsoft-prod.deb
$ sudo apt-get install apt-transport-https
$ sudo apt-get update
$ sudo apt-get install dotnet-sdk-2.2
</code></pre>

Installation is done, to test your work, run this command in the same installation directory:
<pre><code>
$ dotnet --info
</code></pre>

## Now lets install .Net core 3.0 Preview
Go to this folder
<pre><code>
$ cd /usr/share/dotnet
$ sudo wget https://download.visualstudio.microsoft.com/download/pr/498b8b41-7626-435e-bea8-878c39ccbbf3/c8df08e881d1bcf9a49a9ff5367090cc/dotnet-sdk-3.0.100-preview9-014004-linux-x64.tar.gz
$ sudo tar -xvzf dotnet-sdk-3.0.100-preview9-014004-linux-x64.tar.gz
</code></pre>


It’s done, to check your work, type:
<pre><code>
$ dotnet --info
</code></pre>

## Apache2 server Configuration 
In your ubuntu machine, lets do some configuration on apache2 server
<pre><code>
$ sudo a2enmod proxy proxy_http proxy_html proxy_wstunnel
$ sudo a2enmod rewrite
</code></pre>

## .Net Core project Configuration 
Now we will do some configuration to host "ApiService"
<pre><code>
$ sudo nano /etc/apache2/conf-enabled/netcore.conf
</code></pre>
add these lines to the new file
<pre><code>
<VirtualHost *:80>  
   ServerName www.DOMAIN.COM  
   ProxyPreserveHost On  
   ProxyPass / http://127.0.0.1:5000/  
   ProxyPassReverse / http://127.0.0.1:5000/  
   RewriteEngine on  
   RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]  
   RewriteCond %{HTTP:CONNECTION} Upgrade$ [NC]  
   RewriteRule /(.*) ws://127.0.0.1:5000/$1 [P]  
   ErrorLog /var/log/apache2/netcore-error.log  
   CustomLog /var/log/apache2/netcore-access.log common  
</VirtualHost> 
</code></pre>

Change the "ServerName" to your server name of just add the server IP address
Save your work and exit the file
<pre><code>
$ sudo service apache2 restart
$ sudo apachectl configtest
</code></pre>
If your work is ok, you will get an OK Msg

## Enable .Net core application in Apache2 server
Let’s now enable .NET Core project in Apache
First we need to move the published folder in windows machine "Publish folder" to the Ubuntu machine, you can use many tools, in my case i uploaded to my one drive and downloaded to my Ubuntu.
The download folder will get "Publish.zip", unzip it to a pre created folder named "Publish" in the same directory
<pre><code>
$ cd Downloads
$ mkdir Publish
$ unzip Publish.zip -t Publish
</code></pre>

Enable .Net core application in Apache2 server
<pre><code>
$ sudo cp -a ~/Publish/ /var/netcore/
</code></pre>
Go back to the root folder
<pre><code>
$ cd ~
$ sudo nano /etc/systemd/system/kestrel-netcore.service
</code></pre>
Paste these lines to the new opened file
<pre><code>
[Unit]
Description=ASP.NET Web Application
[Service]
WorkingDirectory=/var/netcore
ExecStart=/usr/bin/dotnet /var/netcore/*.dll
Restart=always
RestartSec=10
SyslogIdentifier=netcore-demo
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
[Install]
WantedBy=multi-user.target
</code></pre>

Change this
ExecStart=/usr/bin/dotnet /var/netcore/*.dll

To be this
ExecStart=/usr/bin/dotnet /var/netcore/ApiService.dll

Save and exit 

one last thing
<pre><code>
$ sudo systemctl enable kestrel-netcore.service
$ sudo systemctl start kestrel-netcore.service
</code></pre>

Enjoy .....!
