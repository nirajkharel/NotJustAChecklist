One of the most frequently acknowledged vulnerabilities in bug bounty programs for Android and iOS apps is related to DeepLink, which can be exploited remotely. However, it should be noted that attacking a deeplink schemas is not a straightforward work. There are some technical terms which we need to be explored inorder to attack the deeplink successfully.

**What is a deeplink?**

A deep link in mobile application is like a special link that takes you directly to a specific part of a mobile app instead of a website. This means you can easily switch between apps or go from a website to an app without having to click a lot of buttons. It makes it quicker and easier to find what you're looking for in the app. 

## Android
**Android usually have two type of deeplinks:**
- **Explicit DeepLinks**
  - The technique of using explicit deep links involves displaying dynamic content to users through deep link URLs. By using explicit deep links, the user can be directed to different destinations based on their actions. For example, the user may be directed to destination A via an explicit deep link on their first visit, and to destination B on their second visit, based on their interaction with the application.

- **Implicit DeepLinks**
  - Implicit deep linking is a technique that enables users to access a specific location within a mobile application. This location is predetermined and remains fixed, meaning that the content displayed within the application will remain the same every time the user accesses it via the implicit deep link. For example, if a user watches a YouTube video on their Chrome browser and then opens it in the YouTube application using an implicit deep link, the content of the video will not change.

Watch these videos to understand more about Implicit and Explicit Deeplinks
  - https://www.youtube.com/watch?v=pY9FOqk_pe4
  - https://www.youtube.com/watch?v=XJgPIeolJu8
  
**And those DeepLinks can be used using three different ways:**
- **Scheme URLs** 
  - The introduction of scheme URLs prioritized flexibility over security, as any application can create custom scheme URLs without having to claim a specific one. In cases where multiple applications are available to handle a deep link, the mobile device presents the user with options to choose from.
  - We can identify the Scheme URLs in an application from the AndroidManifest.xml file. Let's take an example of an application [InsecureShop](https://github.com/optiv/InsecureShop/releases/download/v1.0/InsecureShop.apk)
    ```bash
    wget https://github.com/optiv/InsecureShop/releases/download/v1.0/InsecureShop.apk
    apktool d InsecureShop.apk
    vim AndroidManifest.xml
    ```
  ![image](https://user-images.githubusercontent.com/47778874/219850400-96732f54-3726-457e-a4bf-92c2e9a3bc44.png)

  - Here wew can see that the `host` and `schemas` for the DeepLink is defined in AndroidManifest.xml file. We can reconstruct the DeepLink URL from this. Usually application has four different components in an application with two optionals. `SCHEMAS`, `HOST` where `PATH` and `PARAMETERS` are optional.
    ```bash
    com.insecureshop://insecureshop/
    # Or, with PATH and PARAMETERS, a deeplink would look like
    com.insecureshop://insecureshop/profile?id=
    ```
  - This means there is no specific rule to implement that the above DeepLink should be associated with the `insecure shop` application only. Any other application with `cominsecureshop://` schemas defined can handled this. With the same reason it can also arise some vulnerabilities like Link Hijacking which will be explained below.
  - This type of deeplink can have schemas specified as `http`, `https` or any custom URL schemes like `demo://`, `test://` or anything.

- **App Links**
  - App Links are the ways to implement the DeepLinks on an Android and iOS application which has been verified for a website, which means any applicaton can have their specific defined App Link which cannot be handled by other applications. App Links mantdatorily requires the `Schemas` value to start with `http` or `https` on a deeplink URLs. It is supported from Android version 6.0 higher and iOS 9 and higher.
  - For a App Link to work successfully, it is required that the app is approved for the specific domain contained in that web intent. If your app isn't approved for the domain, the web intent resolves to the user's default browser app instead.
  - A JSON verification file needs to be available at `<schemas>://<host>.well-known/assetlinks.json` containing the application's package name and keystore fingerprint.
  - We can verify the App Linking via below example. Download a [demo application](https://simonmarquis.github.io/Android-App-Linking/) and view the AndroidManifest.xml file. We can see that the DeepLink schemes has been defined.
 ![image](https://user-images.githubusercontent.com/47778874/219855867-f338591e-fa74-45d6-a234-c35f33dcaf58.png)


  - We can reconstruct the DeepLink URLs from above image
    ```bash
    https://smarquis.fr/action
  
    # https://smarquis.fr/.well-known/assetlinks.json
    ```
  - We can verify the association of the application for the mentioned deeplink by navigating URL `https://smarquis.fr/.well-known/assetlinks.json` into a browser which will result like:
  ![image](https://user-images.githubusercontent.com/47778874/219856058-c9485f97-777c-473c-90cf-6fe539ee0d4d.png)

- **Intent URLs** 
  - The Deep Link type known as Intent URLs is not commonly used in mobile applications. It includes the package name declaration in its Deep Link schemas, which allows the application to be opened directly without requiring the user to specify the application when they click the link.

### Deep Link Enumeration

There are various kind of vulnerabilities which can be found on the Deep Link which are discussed below. To identify and exploit such vulnerabilities, we first need to identify the Deep Link schemas. Let's enumerate the Deep Link at first. There are various ways where a DeepLink can be configued on an application.

**Identify DeepLink URIs 1**
- Sometimes the application only contains the deeplink schemas and does not requires any other components like HOST, PARAMETERS. We can check such schemas manually.
- [Example APK](https://m.apkpure.com/injuredandroid/b3nac.injuredandroid/download)
- View Android Manifest file
  ```bash
  wget https://d.apkpure.com/b/APK/b3nac.injuredandroid?version=latest
  apktool d InjuredAndroid_1.0.9_Apkpure.apk
  cd InjuredAndroid_1.0.9_Apkpure
  cat AndroidManifest.xml
  ```
- We can find the deeplink schemas defined on the Manifest File.
  ```xml
      <activity android:label="@string/title_activity_deep_link" android:name="b3nac.injuredandroid.DeepLinkActivity">
            <intent-filter android:label="filter_view_flag11">
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="flag11"/>
            </intent-filter>
            <intent-filter android:label="filter_view_flag11">
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="https"/>
            </intent-filter>
      </activity>
  ```
- It always does not needs `HOST` to create a DeepLink. Here we have an `SCHEMA` only. Let's open the JAVA file corresponding to the activity defined `b3nac.injuredandroid.DeepLinkActivity.java`
  ```bash
  # Decompile the APK
  - jadx /home/username/InjuredAndroid_1.0.9_Apkpure.apk -d /home/username/jadx-decompiled
  - vi jadx-decompiled/sourcs/b3nac/injuredandroid/DeepLinkActivity.java
  ```
- On the Java File, we can see that the application only checks where the intent `android.intent.action.VIEW` have the data `flag11` as the scheme. It does not contains further components and checks only if scheme is valid which is `flag11`.
  ```java
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_deep_link);
        j.g.a(this);
        Intent intent = getIntent();
        d.m.b.d.b(intent, "intentToUri");
        Uri data = intent.getData();
        if (d.m.b.d.a("flag11", data != null ? data.getScheme() : null)) {
            startActivity(new Intent("android.intent.action.VIEW"));
        }
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
    }
  ```
- Therefore the deeplink URI for this application would only be `flag11://`


**Identify DeepLink URIs 2**
- Sometimes there might be a DeepLink configured which requires just `Schemas` and `Host`. Let's take an example of [AndroGoat](https://github.com/satishpatnayak/AndroGoat)
- Download the application and decompiled it with `apktool`. On viewing the DeepLink schemas, we can find:
  ```XML
        <activity android:label="@string/activity" android:name="owasp.sat.agoat.AccessControl1ViewActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <data android:host="vulnapp" android:scheme="androgoat"/>
            </intent-filter>
        </activity>

  ```
- Decompile the application again with `JADX` to view the source code in Java.
- Navigate to `/sources/owasp/sat/agoat/` and Open the file `AccessControl1ViewActivity.java`
  ```java
      public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_access_control1_view);
        Button downloadInvoice = (Button) findViewById(R.id.downLoad);
        downloadInvoice.setOnClickListener(new View.OnClickListener() { // from class: owasp.sat.agoat.AccessControl1ViewActivity$onCreate$1
            @Override // android.view.View.OnClickListener
            public final void onClick(View it) {
                AccessControl1ViewActivity.this.startService();
            }
        });
    }

    public final void startService() {
        Intent ServiceIntent = new Intent(this, DownloadInvoiceService.class);
        startService(ServiceIntent);
    }
  ```
- We can see that the above code does not query for any parameters, which makes our DeepLink URI as `androgoat://vulnapp` 

**Identify DeepLink URIs 3**
- Sometimes application defines Deep Link Schemas, Host, Path and Parameter as well. In such case we have to identify all the required components to build the DeepLink URIs. I have created a simple application for this. Let's decompiled it and view the Manifest file.
  ```XML
    <activity android:exported="true" android:label="@string/app_name" android:name="com.vulnerable.deeplink.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:host="deeplink" android:pathPrefix="/demo" android:scheme="vulnerable"/>
            </intent-filter>
       </activity>
  ```
- Let's view the Java file correspondence to the activity where the deeplink was defined.
- The file would be under com/vulnerable/deeplink/MainActivity.java when decompiled from JADX
  ```Java
        public class MainActivity extends AppCompatActivity {
          /* JADX INFO: Access modifiers changed from: protected */
         
          public void onCreate(Bundle savedInstanceState) {
              String test3;
              super.onCreate(savedInstanceState);
              setContentView(R.layout.activity_main);
              Uri data = getIntent().getData();
              if (data != null && (test3 = data.getQueryParameter("whoami")) != null) {
                  Intent intent = new Intent("android.intent.action.VIEW", Uri.parse(test3));
                  startActivity(intent);
              }
          }
      }
  ```
- Here we can see that the application needs a parameter called `whoami` and then only the deeplink can be executed.
- In this case our Deep Link URI would be
  ```bash
  vulnerable://deeplink/demo?whoami=
  ```
- Thefore we should also view the Java file where the corresponding activity is defined to get the full Deep Link Uri.
- The parameter needed for the deeplink are also stored on the `Strings.xml` file. We should always check the `/res/values/Strings.xml` file as well.
- We can also sometimes get the deeplink parameter from the source code. In such can we can do the grep as well. 
  - `grep -iRn vulnerable://deeplink`

**Identify DeepLink URIs - DeepLink Fuzzing**



## iOS

