<div align="center">
<h1> 📵 Android Penetration Testing :nepal: </h1>
<a href="https://twitter.com/nirajkharel7" ><img src="https://img.shields.io/twitter/follow/nirajkharel7?style=social" /> </a>
</div>

# Contents  
**1. [Setup and Decompile](#setup-and-decompile)**  
**2. [Verify Signing](#verify-signing)**  
**3. [Check for Hardcoded and URL endpoints](#check-for-hardcoded-and-url-endpoints)**  
**4. [Digging into AndroidManifest.xml File](#digging-into-androidmanifest-file)**  
**5. [Network](#network)**  
**6. [Storage](#storage)**    
**7. [Analyze the source code thoroughly specially on](#analyze-the-source-code-thoroughly-specially-on)**  
**8. [Reverse Engineering](#reverse-engineering)**  
**9. [Cryptography](#cryptography)**   
**10. [Exploiting backup](#exploiting-backup)**   
**11. [Exploiting Android Debuggable](#exploiting-android-debuggable)**   
**12. [Exploiting Exported Components with drozer](#exploiting-exported-components-with-drozer)**  
**13. [Intercepting HTTPS request on Java Based Application](#intercepting-https-request-on-java-based-application)**  
**14. [Intercepting HTTPS request on Flutter Based Application](#intercepting-https-request-on-flutter-based-application)**   
**15. [Testing APIs](#testing-apis)**   
**16. [References](#references)**   
**17. [More Resources](#more-resources)**   

## Setup and Decompile
- Install the application on emulator or physical device
- View the package name of application with ADB
  - `adb shell pm list packages | grep "app-name"`
- View the path of the application.
  - `adb shell pm path "package-name"`
- Decompile the application with APKTOOL
  - `apktool d app.apk`
- Decompile the application with JADX
  - `jadx /path/to/apk -d /path/to/output`

## Verify Signing
- [ ] Check if the application has been properly signed.
```bash
   apksigner verify --verbose app.apk
   jarsigner -verify -verbose -certs app.apk
```
## Check for Hardcoded and URL endpoints
- [ ] Hardcoded Emails
  - ` grep -HnroE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b' | awk -F: '{print $3}'`
- [ ] Hardcoded AWS Keys
  - ` gf aws-keys`
- [ ] Hardcoded IPs
  - ` gf ip`
- [ ] Hardcoded Base64 data
  - ` gf base64`
- [ ] Hardcoded usernames and passwords on the source code
- [ ] Hardcoded data on strings.xml ➡️ /res/values/strings.xml
- [ ] Hardcode encryption keys and secrets
- [ ] Hardcoded data on raw.xml ➡️ /res/values/raw.xml
- [ ] Hardcoded URL endpoints
  - ` gf urls`

## Digging into AndroidManifest File
- [ ] Check if backup is enabled ➡️ `android:allowBackup="true"`
- [ ] Check if debug mode is enabled ➡️ `android:debuggable="true"`
- [ ] DeepLink ➡️ Check for deeplink schemas
- [ ] Permission ➡️ Check if application is asking unnecessary permission.
- [ ] Permission ➡️ Check if application is having permission like External Storage, SMS
- [ ] Check the exported activites, services, providers and receivers
- [ ] Intent Filter also makes components exported by default
- [ ] Task Hijacking ➡️ Check if `launchMode=singleTask`defined
- [ ] Check the Implicit and Explicit Intents
- [ ] Note down the interesting activites on the manifest file
- [ ] Check if HTTP is enabled ➡️ `cleartextTrafficPermitted="true"`

## Network 
- [ ] Check the Network Security config file ➡️ `/res/xml/network_security_config.xml`
- [ ] URL with HTTP protocol on the source code
  - Find the URLs with `gf url` and list the HTTP only
  - Or, with GREP ➡️ `grep -iRn "http://"`
- [ ] Check if SSL pinning is enabled on the application or not

## Storage
- [ ] Shared Preferences
  - [ ] Check if the shared preferences contains sensitive data unecrypted ➡️ `/data/data/package-name/sharedprefs`
  - [ ] World Writable - Check if the shared preferences is world writable
  - [ ] World Readable ➡️ Check if the shared preferences is world readable
  - MODE_WORLD_READABLE allows all applications to access and read the content of the file
  ```java
    SharedPreferences sharedPref = getSharedPreferences("key", MODE_WORLD_READABLE);
    ```
- [ ] SQLite Database
  - [ ] Check if database contains sensitive information unencrypted ➡️ `/data/data/package-name/files/`
- [ ] Firebase
  - [ ] Check for firebase URL endpoint with `gf url` or on `/res/values/strings.xml`
  - [ ] Add /.json on the end of the URL `curl -X GET https://subdomain.firebaseio.com/.json`
    - The endpoint will be vulnerable if it does not replies with `"error" : "Permission denied"`
- [ ] Internal Storage
  - [ ] World Writable - Check if the internal storage is world writable
  - [ ] World Readable - Check if the external storage is world readable
  ```java
  fos = openFileOutput(FILENAME, Context.MODE_PRIVATE);
  ```
  - [ ] Analyse the data in internal storage
    - Pull the internal storage without using (just after installation)
    `adb pull /data/data/package-name`
    - Pull the internal storage after using (full feature usage)
    `adb pull /data/data/package-name`
    - Analyze the difference
- [ ] External Storage
  - Application can store data externally on SD card or on internal (non-removal).
  - Pull the external storage of the application and look for any sensitive information.
  ```bash
  # locate the external storage of the app
  adb shell ls /sdcard/Android/data
  
  # Pull the external directory of the app
  adb pull /sdcard/Android/data/"package-name"
  ```
  
  - File saved to external storage is World Readable which can be accessed by other apps.
  ```java
  File file = new File (Environment.getExternalFilesDir(), "password.txt");
  ```
  - Search for Classes and Functions such as
    - `MODE_WORLD_READABLE` or `MODE_WORLD_WRITABLE`
    - the `SharedPreferences` class (stores key-valu pairs)
    - the `FileOutPutStream` class (uses internal or external storage)
    - the `getExternal*` functions(use external storage)
    - the `getWritableDatabase` function (returns a SQLiteDatabase for writing)
    - the `getReadableDatabase` function (returns a SQLiteDataabase for reading)
    - the `getCacheDir` and `getExtrernalCacheDirs` function (use cached files)
  - [ ] Check for HTTP caching (sensitive information) which is stored in Cache.db in internal storage
  - [ ] Keyboard Cache
    - Determine Whether the Keyboard Cache is Disabled for Sensitive Input.
    - The keyboard cache file is located in: `/var/mobile/Library/Keyboard`
    ```xml
    <EditText
        android:id="@+id/KeyBoardCache"
        android:inputType="textNoSuggestions" />
    ```
## Analyze the source code thoroughly specially on
- [ ] Login and Registration
- [ ] Password Forgot and Reset
- [ ] Activity which has implemented the intent defined on AndroidManifest.xml
- [ ] Encryption and Decryption
- [ ] Web API Intregation
- [ ] Transactions Related
- [ ] OTP Related
- [ ] Activity which has implemented webviews
- [ ] Database, Shared Preferences, Internal and External Directory Integration
- [ ] Build Configs


## Reverse Engineering
- [ ] Decompile
  - Basic Decompilation
    - `apktool d base.apk`
  - Decompile only resource
    - `apktool d -s base.apk`
  - Decompile only source
    - `apktool d -r base.apk`
  - If above give errors
    - `apktool --use-aapt2 d base.apk`
- [ ] Recompile
  - `apktool b --use-aapt2 -o out.apk base`
- [ ] Generate a key
  - `keytool -genkey -v -keystore keys/test.keystore -alias Test -keyalg RSA -keysize 1024 -sigalg SHA1withRSA -validity 10000`
- [ ] Sign the application
  - `jarsigner -keystore keys/test.keystore dist/test.apk -sigalg SHA1withRSA -digestalg SHA1 Test`
- [ ] Zipalign the application
  - `zipalign -v 4 <YOUR_APP_NAME> app-release-final.apk`
- [ ] Automate the signing and zipalign with [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer/releases/tag/v1.2.1)
  - `java -jar uber-apk-signer-1.2.1.jar -a out.apk`

## Cryptography
- [ ] Check if application is communicating with server encrypted or unencrypted
- [ ] Check if the private key secrets are being stored hardcode on the source code
- [ ] Check if the application is using Base64 for encrypting data for transmitting or storing ➡️ `gf base64`
- [ ] Check for depreciated algorithms like RC1, MD4, MD5 and SHA1 which causes collision issues
- [ ] Analyze the source code and workflow of the encryption technique
  - Sometimes there might not be issues on hardcoding secrets, depreciated algorithm, but the implementation of the techniques might be done insecurely
- [ ] Check if the application has implemented their own custom cryptographic algorithm and try to reverse it.

## Exploiting backup
- [ ] Check if application has backup enabled. If enabled, use following commands to extract internal storage even if the device is non-rooted.
```bash
   # Look for backup on Manifest file
   android:allowBackup="true"
   
   # List the package name
   adb shell pm list packages | grep "app name"
   
   # Backup the data contained in the package.
   adb backup "package-name"
   
   # The file name will be backup.ab
   # Convert .ab extension with android backup extractor.
   Download link: https://sourceforge.net/projects/adbextractor/
   
   # Convert .ab into tar
   java -jar abe.jar unpack backup.ab backup.tar
   
   # Extract the tar file
   tar -xvf backup.tar
   
   # Analyze the files in the extracted folder and look for directory like databases, shared preferences, files
```
## Exploiting Android Debuggable
- [ ] Check if application has debuggable mode enabled. If enabled, use following commands to access the android shell as the vulnerable application even if the device is not rooted.
```bash
   # Look for debuggable mdoe on Manifest file
   android:debuggable="true"
   
   # Before app start run,
   adb jdwp
   # It will list all the process ID of the running applications.
   
   # After app start run,
   adb jdwp
   # The currently started application process ID can be identified.
   
   adb shell ps | grep "ID-number"
   # It will give the package name of the ID.
   
   adb shell
   run-as package-name
   
   # Or,
   adb shell pm list packages | grep "app-name"
   adb shell
   run-as package-name
```
- [ ] For further exploitation, follow [Link1](https://securitygrind.com/how-to-exploit-a-debuggable-android-application/) and [Link2](https://resources.infosecinstitute.com/topic/android-hacking-security-part-6-exploiting-debuggable-android-applications/).

## Exploiting Exported Components with drozer
**Setup**
- [ ] Download drozer app for android and PC from [here](https://github.com/FSecureLABS/drozer).
- [ ] Install the applications on android and PC.
- [ ] Forward the port from PC
```bash
   adb forward tcp:31415 tcp:31415
```
- [ ] Open the drozer application on android device and click on ON button
- [ ] Launch the application
```bash
   drozer console connect
```

**Retrieving Package information**
- [ ] View the package name of the application
```bash
   run app.package.list -f "app-name"
```
- [ ] View the basic information of the package
```bash
   run app.package.info -a "package-name"
```
- [ ] View the Manifest information of the package
```bash
   run app.package.manifest "package-name"
```
- [ ] Identify the attack surface
```bash
   run app.package.attacksurface "package-name"
   # It will list all the exported activities, services, providers, broadcasts.
```
**Exported Activities**
- [ ] Verify if unnecessary activities are exported and exploit it
```bash
   run app.activity.info -a "package-name"
   # It will list the exported activities
   
   run app.activity.info -a "package-name" -u
   # It will list all the activities
   
   # Start the exported activity from drozer and see the result on android device
   run app.activity.start --component "package-name" "component-name"
```
**Exported Content Providers**
- [ ] Verify if any content providers are exported and exploit it
```bash
   run app.provider.info -a "package-name"
   # It will list the exported providers
   
   run scanner.provider.finduris -a "package-name"
   # It will list the URIs of the exported providers
   
   run app.provider.query "uri"
   # List the query on exported URIs
   
   run app.provider.update "uri" --selection <conditions> <selection arg> <column> <dataa>
   # Update the database using exported URIs
```
- [ ] SQLi on Content providers
```bash
   run scanner.provider.sqltables -a "package-name"
   # List the tables
   
   run scanner.provider.injection -a "package-name"
   # Performs some basic SQLi
```
- [ ] Path traversal on Content providers
```bash
   run scaner.provider.traversal -a "package-name"
```
**Exported Broadcast Receivers**
- [ ] Verify any exported broadcast receivers and exploit it
```bash
  run app.broadcast.info -a "package-name"
  # List the exported broadcast receivers
  
  run app.broadcast.send --component "package-name" "component-name" --extra <type> <key> <value>
  # Send data through exported broadcast receivers
  
  run app.broadcast.sniff --action <action>
  # Sniff the data through exported broadcast receivers
```
**Exported Services**
- [ ] Verify any exported services and exploit it
```bash
  run app.service info -a "package-name"
  # List the exported services
  
  run app.service.start --action <action> --component <package-name> <component-name>
  # Start the exported services
  
  run app.service.send <package-name> <component-name> -msg <what> <arg1> <arg2> --extra <type> <key> <value> --bundle-as-obj
  # Send data through services
```

## Intercepting HTTPS request on Java Based Application
**With Frida**
- [ ] Download and install frida
```bash
pip3 install frida-tools
```
- [ ] Download frida servers according to your android architecture from [here](https://github.com/frida/frida/releases) and rename it to frida-server
```bash
   # View CPU architecture of the android device
   adb shell getprop ro.product.cpu.abi
```
- [ ] Download the certificate from burp and copy as cert-der.crt
- [ ] Push the server and certificate file to android
```bash
   adb root # might be required
   adb push frida-server /data/local/tmp/
   adb push cert-der.crt /data/local/tmp/
   adb shell "chmod 755 /data/local/tmp/frida-server"
   adb shell "/data/local/tmp/frida-server &"
```
- [ ] Download the SSL Pinning bypass script from [here](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/) and save it as sslPinningBypass.js
- [ ] Change proxy on android device
  - Long press on wifi
  - Click on Advance option
  - Change proxy to manual
  - Enter your PC ip address
  - Enter a random port (like 5567)
  - Save
- [ ] Open the burp and Change the proxy option from 127.0.0.1:8080 to PC-IP:5567 (port number must be same)
- [ ] Grab the package name with `adb shell pm list packages | grep "app-name"`
- [ ] Run the script
```bash
   frida -U -l sslPinningBypass.js -f "package-name" --no-pause
```
- [ ] The application will open automatically and now the requests will be captured on burp

**With Objection (works also on non-rooted device)**
- [ ] Install Objection
```bash
   pip3 install objection
```
- [ ] Patch the apk with Objection
```bash
   objection patchapk -s base.apk
```
- [ ] If the above patch failed, follow [here](https://fadeevab.com/frida-gadget-injection-on-android-no-root-2-methods/) for manual patch of application using objection
- [ ] Installed the patched apk
```bash
   adb install patched.pak
```
- [ ] Proxy the android route into PC as described on frida section
- [ ] Setup burp as described on frida section
- [ ] Run the Application
- [ ] Run Objection explore
```bash
   objection explore
```
- [ ] Disable SSL Pinning
```bash
   android sslpinning disable
```
- [ ] Burp will start intercepting the packets

## Intercepting HTTPS request on Flutter Based Application
The use of Frida on bypassing SSL Pinning may not work on flutter. I also performed some exploitatio in libflutter.so file for injecting the memory address of the certificate checker. However it did not work as intended.
[Here](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/) is the reference for bypassing SSL Pinning on flutter application through the exploitation of libflutter.so file.

This is an alternative method for bypassing SSL Pinning in Flutter application.   
***Note: I have tested this method on few aplications with success but this might not work as well.***
- [ ] Requirements
  - Genymotion: Android Emulator
  - ProxyDroid: Since flutter appication ignores manual proxy configuration through wifi
  - Burp Suite
- [ ] Setup and Configurations
  - Install ARM Translation tool on the device since flutter application and proxydroid may not work in genymotion since it is of x86 architecture.
  - Click on the **File Upload** widget and **INSTALL OPEN GAPPS**
  - Click **INSTALL**
  - Wait for the package to be installed
  - When done, you will be prompted to reboot the virtual device.
  - (Optional) Open the Play Store and configure you account.
  - Verify ARM Translation tools installtion
  ```bash
     adb shell getprob ro.product.cpu.abilist
     # This command should return "x86,armeabi-v7a,armeabi" as the output.
  ```
- [ ] Download the Burp Certificate
- [ ] Convert Burp Certificate into PEM format
  - Android only accepts certificate of PEM format if needs to be installed as system certificate.
  - We can convert the burp certificate into PEM format using openssl. [More](https://gist.github.com/PaulSec/dab5d25573d7f2d7da18)
  ```bash
     openssl x509 -inform DER -in cacert.der -out cacert.pem
     openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1

     # Rename the PEM file with the output given by the second command
     mv cacert.pem <RandomAlphaNumeric>.0

     # Example: mv cacert.pem 4a787b67.0
  ```
- [ ] Push the certificate file into android application
  - We need to push the certificate into /system/etc/security/cacerts directory inorder to install it as system certificate. But although having root access on the device, permission to write on system directory is denied. For this we need to remount the device.
  ```bash
     adb root
     adb remount

     # Push the file
     adb push 4a787b67.0 /system/etc/security/cacerts/

     # If the above command still throws error. Consider pushing it into the sdcard at first.
     adb push 4a787b67.0 /sdcard/Download/
     adb shell
     mv /sdcard/Download/4a787b67.0 /system/etc/security/cacerts/

     # Change the permission of the file (inside adb shell)
     chmod 644 /system/etc/security/cacerts/4a787b67.0

     # Reboot the device (inside adb shell)
     reboot
  ```
  - Verify the certicate installation by navigating to installed certificates on android device through settings. Should list Portswigger CA.
 - [ ] Install and Configure the ProxyDroid aplication on the device
  - Since the flutter application ignores the proxy configured through wifi options, proxydroid must be installed.
  - Download Link is [here](https://proxydroid.en.uptodown.com/android)
  - Open the application
    - Enter the IP address of your computer (which intercepts the traffic) on the host parameter and a random port number and enable the proxy.
    - Remove the custom proxy from the Wifi advance option if you have configured previously.
 
- [ ] Burp's Things
  - Open the Burp
  - Go To: Proxy ➡️ Options ➡️ Add ➡️ Request handling
  - Enter the previously entered host and port from proxy droid
  - Click on Support invisible proxying.

- [ ] Save the configuration, and you will be able to intercept the traffic of the flutter application
  
## Testing APIs
- [ ] Please follow [this](https://github.com/nirajkharel/NotJustAChecklist/blob/main/API.md) for API Pentesting Checklist

## References
- [OWASP MSTG](https://github.com/OWASP/owasp-mstg)
- [Hacktrick Android Application Penetration](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting)

## More Resources
- [Exploiting Android deep links and exported components by b3nac](https://www.youtube.com/watch?v=lg1sN8njSYs&t=1344s)
