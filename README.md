# Azure Naming Tool

<img src="./src/wwwroot/images/AzureNamingToolLogo.png?raw=true" alt="Image representing the Azure Naming Tool" title="Azure Naming Tool" height="150"/>

## Overview

The Azure Naming Tool was created to help administrators define and manage their naming conventions, while providing a simple interface for users to generate a compliant name. The tool was developed using a naming pattern based on [Microsoft's best practices](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging). Once an administrator has defined the organizational components, users can use the tool to generate a name for the desired Azure resource.

## [CLICK HERE FOR DOCUMENTATION!](https://github.com/mspnp/AzureNamingTool/wiki)

## How to run it on ubuntu server with apache2
- First install dotnet: `sudo apt update && sudo apt install dotnet-sdk-10.0 apache2 -y`
- Download the **AzureNamingTool.zip** file from [Release page](https://github.com/mspnp/AzureNamingTool/releases)
- Extract the folder to /var/www/ and build the project
```
sudo unzip AzureNamingTool.zip -d /var/www/aznaming
cd /var/www/aznaming
sudo dotnet clean
sudo dotnet build
sudo mv /var/www/aznaming/azurenamingapp/wwwroot/_content/Blazored.Modal/BlazoredModal.razor.js /var/www/aznaming/azurenamingapp/wwwroot/_content/Blazored.Modal/BlazoredModal.js
sudo dotnet build
```
- Create a Service for Azure Naming Tool
`sudo nano /etc/systemd/system/aznaming.service`
- Paste the content below in the aznaming.service
```
[Unit]
Description=Azure Naming Service

[Service]
WorkingDirectory=/var/www/aznaming/
ExecStart=/usr/bin/dotnet run --project /var/www/aznaming/AzureNamingTool.csproj
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=InspectorGadget-identifier
User=root
Environment=ASPNETCORE_ENVIRONMENT=Development
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```
- Create a virtual directory for the Azure Naming tool:
- Run this command: `sudo nano /etc/apache2/sites-available/aznaming.conf`
- Then paste the following content into /etc/apache2/sites-available/aznaming.conf - replace ServerName/ServerAlias with your Custom DNS or with localhost:
```
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5222/
    ProxyPassReverse / http://127.0.0.1:5222/
    ServerName aznaming.contoso.com
    ServerAlias www.aznaming.contoso.com
    ErrorLog ${APACHE_LOG_DIR}/aznaming_error.log
    CustomLog ${APACHE_LOG_DIR}/aznaming_access.log combined
</VirtualHost>
```
- Then run the following commands:
```
sudo a2ensite aznaming.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo chown -R www-data:www-data /var/www/aznaming
sudo chmod -R 755 /var/www/aznaming
sudo systemctl restart apache2
```
- Then start the aznaming service
```
sudo systemctl start aznaming
sudo systemctl status aznaming
```
- Now try to access the Azure Naming website using your http://aznaming.contoso.com or http://localhost or http://ServerIPAddress
