<div align="center">
<h1> 📵 IOS Penetration Testing :nepal: </h1>
<a href="https://twitter.com/nirajkharel7" ><img src="https://img.shields.io/twitter/follow/nirajkharel7?style=social" /> </a>
</div>

# Contents  
**1. [Setup](#setup)**  
**2. [Cydia Configurations](#cydia-configurations)**  
**3. [SSH into Device](#ssh-into-device)**  
**4. [Extracting the IPA File](#extracting-the-ipa-file)**  
**5. [Decompile the IPA file](#decompile-the-ipa-file)**  
**6. [Check for Hardcoded and URL endpoints](#check-for-hardcoded-and-url-endpoints)**    
**7. [Digging into FileSystem in IOS](#digging-into-filesystem-in-ios)**  
**8. [Intercepting HTTP Traffic on IOS](#intercepting-http-traffic-on-ios)**  
**9. [Analyze the source code thoroughly specially on](#analyze-the-source-code-thoroughly-specially-on)**   
**10. [Cryptography](#cryptography)**     
**11. [Testing APIs](#testing-apis)**   
**12. [References](#references)**   
**13. [More Resources](#more-resources)**   

## Setup 
### Jailbreak the ios device.
- [ ] Download and install [checkra1n](https://checkra.in) on MAC
- [ ] Connect the ios device with MAC through USB
- [ ] Open the checkra1n app and proceed further. (If the process failed, click on options button and check allow untrusted
- [ ] Continue, the device will move into recovery mode.
- [ ] Press and hold lock and volume down button and release lock button after 4 seconds. But continue pressing volume down button.
- [ ] The device will start jailbreaking itself.

## Cydia Configurations
### Cydia can be used to install different packages on jailbroken device.
* Some of the apps which should be installed from cydia
  - [ ] Frida
  - [ ] SSH
  - [ ] Filza

## SSH into device
- [ ] ssh root@<device-ip>. Password is `alpine`.
- [ ] Forgot root password
  * Use Filza to edit the `/private/etc/master.passwd`. This file contains hashes to all passwords of iOS users. Find the root record and modify it to the following value:
`root:/smx7MYTQIi2M:0:0::0:0:System Administrator:/var/root:/bin/sh`
- [ ] It will reset the password to `alpine`.
  
## Extracting the IPA File
* The IPA file installed from App Store is encypted. We can decrypt it by using frida.
  - [ ] Download and install frida on PC. `pip3 install frida-tools`
  - [ ] Grab the Display name or bundle identifier of the application using `frida-ps -Uai`
  - [ ] Download a python file which is built using frida to dump the IPA from [here](https://github.com/AloneMonkey/frida-ios-dump).
  - [ ] Install the requirements
  - [ ] Forward SSH port with Iproxy `iproxy 2222 22`
  - [ ] Run `python3 dump.py "Display Name"`
  - [ ] If the script does not start dumping itself. Open the app and keep the script running on background.

## Decompile the IPA file
- [ ] Decompile using unzip. The decompiled directory will be named as `Payload`
  - `unzip appname.ipa`
- [ ] Application Structure
  - .plist - A file which contains specific configurations. (Example: Info.plist)
  - Frameworks - A folder which contains native libraries.
  - PlugIns - A folder which contains app extensions.
  - Assets.car - A compressed folder which contains resources. (Example: icons, images)
  - Appname - A binary executable file which contains actual source code.

## Check for Hardcoded and URL endpoints
- [ ] Hardcoded Emails
  - From whole decompiled directory: `cd Paylod/Appname.app/`
  - ` grep -HnroE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b' | awk -F: '{print $3}'`
  - From binary executable file: `cd Payload/Appname.app/`
  - `strings Appname | grep -HnroE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b' | awk -F: '{print $3}'`
- [ ] Hardcoded AWS Keys
  - ` gf aws-keys`
- [ ] Hardcoded IPs
  - ` gf ip`
- [ ] Hardcoded Base64 data
  - ` gf base64`
- [ ] Hardcoded URL endpoints
  - ` gf urls`
- [ ] Hardcoded Google API Keys
  - `strings Appname`
  - Download and run Goole Map API Scanner from [here](https://github.com/ozguralp/gmapsapiscanner).
- [ ] Hardcoded usernames and passwords on the source code. `strings Appname`
- [ ] Hardcoded data on Info.plist ➡️ `/Payload/Appname.app/Info.plist`
- [ ] Hardcode encryption keys and secrets
- [ ] Hardcoded data on settings.json ➡️ `/Payload/Appname.app/settings.json`


## Digging into FileSystem in IOS
- IOS applications have two main directories `Bundle` and `Data`
- **Bundle Directory**
  - Contains application binary file, applicationn data and static files.
  - Grab the bundle directory
  - [ ] Connect to application with objection. `objection -g <package_name> explore`
  - [ ] Get the directories list `env`. The bundle folder with displayed something like `/var/containers/Bundle/Application/3ADAF47D-A734-49FA-B274-FBCA66589E67/iGoat-Swift.app`
  - [ ] SSH into the device `ssh root@<device_ip>`
  - [ ] Navigate to the directory and analyze the files on it `cd /var/containers/Bundle/Application/3ADAF47D-A734-49FA-B274-FBCA66589E67/iGoat-Swift.app`
- **Data Directory**
  - Contain the files the developer wants to keep.
  - Files that could be used for caching information for quick access, or storing offline data as a backup for resuming the application use.
  - **Caches Directory**
    - The caches directory contains semi-persistent cached files. It can store sensitive informations and the OS will delete the contains in this directory when the device storage is low and application is not running.
    - [ ] Connect to application with objection and list the internal directories with `env`
    - [ ] SSH into the device
    - [ ] Navigate to cache directory. Example: `cd /var/mobile/Containers/Data/Application/8C8E7EB0-BC9B-435B-8EF8-8F5560EB0693/Library/Caches`
  - **Document Directory**
    - The Document Directory contains all the user generated informations on it.
    - The application user can write into this directory. Example: Downloading transaction receipt on the mobile banking application, the receipt will be stored on this directory.
    - This directory can be accessed by user and does not needed for a device to be jailbroken.
    - [ ] Connect to application with objection and list the internal directories with `env`
    - [ ] SSH into the device
    - [ ] Navigate to cache directory. Example: `cd /var/mobile/Containers/Data/Application/8C8E7EB0-BC9B-435B-8EF8-8F5560EB0693/Documents`
  - **Library Directory**
    - This directory contains the files and folders necessary to run the application and are not user specific which means users cannot modify the contents on this directory until jailbreak.
    - Contains files and directories like plist files, caches, preferences.
    - [ ] Navigate to this directory. Example: `cd /var/mobile/Containers/Data/Application/8C8E7EB0-BC9B-435B-8EF8-8F5560EB0693/Library`
    - [ ] Check for preferences directory inside Library too since it can contain the sensitive informations like username, passwords, PII and much more.

## Intercepting HTTP Traffic on IOS
- [ ] The IOS device and interceptor should be on the same LAN.
- [ ] Navigate to Proxy tab and Add a new listener. The IP address should be of your computer and port can be any. Example: 5567
- [ ] Open the Settings on iOS device and Click on Wifi.
- [ ] Click on `(i)` symbol.
- [ ] Select Manual on HTTP Proxy and enter IP address and PORT configured on the Burp Proxy.
- [ ] The HTTP traffic should be intercepted by the Burpsuite.

## Analyze the source code thoroughly specially on
- [ ] Login and Registration
- [ ] Password Forgot and Reset
- [ ] Plist files
- [ ] Encryption and Decryption
- [ ] Web API Intregation
- [ ] Transactions Related
- [ ] OTP Related
- [ ] Activity which has implemented webviews
- [ ] Database, Internal and External Directory Integration
- [ ] Build Configs

## Cryptography
- [ ] Check if application is communicating with server encrypted or unencrypted
- [ ] Check if the private key secrets are being stored hardcode on the source code
- [ ] Check if the application is using Base64 for encrypting data for transmitting or storing ➡️ `gf base64`
- [ ] Check for depreciated algorithms like RC1, MD4, MD5 and SHA1 which causes collision issues
- [ ] Analyze the source code and workflow of the encryption technique
  - Sometimes there might not be issues on hardcoding secrets, depreciated algorithm, but the implementation of the techniques might be done insecurely
- [ ] Check if the application has implemented their own custom cryptographic algorithm and try to reverse it.
  
## Testing APIs
- [ ] Please follow [this](https://github.com/nirajkharel/NotJustAChecklist/blob/main/API.md) for API Pentesting Checklist

## References
- [HackTricks IOS Pentesting](https://book.hacktricks.xyz/mobile-apps-pentesting/ios-pentesting)
- [OWASP MSTG IOS](https://mobile-security.gitbook.io/mobile-security-testing-guide/ios-testing-guide/0x06a-platform-overview)

## More Resources
- [Understanding IOS FileSystems](https://medium.com/@lucideus/understanding-the-ios-file-system-eee3dc87e455)
- [IOS Pentesting - Cobalt](https://cobalt.io/blog/ios-pentesting-101)
