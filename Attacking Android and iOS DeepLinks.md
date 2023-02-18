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

Here wew can see that the `host` and `schemas` for the DeepLink is defined in AndroidManifest.xml file. We can reconstruct the DeepLink URL from this. Usually application has four different components in an application with two optionals. HOST, SCHEMAS, PATH AND PARAMETERS.
```bash
com.insecureshop://insecureshop/
```

- **App Links** 
- **Intent URLs** 

**Scheme URLs**

**App Links**

**Intent URLs**

## iOS
