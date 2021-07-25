<div align="center">
<h1> 🏹 API Penetration Testing :nepal: </h1>
<a href="https://twitter.com/nirajkharel7" ><img src="https://img.shields.io/twitter/follow/nirajkharel7?style=social" /> </a>
</div>

API Security Top 10 2019
------------------------

-   **API1:2019 Broken Object Level Authorization**

    APIs tend to expose endpoints that handle object identifiers,
    creating a wide attack surface Level Access Control issue. Object
    level authorization checks should be considered in every function
    that accesses a data source using an input from the user.

-   **API2:2019 Broken User Authentication**

    Authentication mechanisms are often implemented incorrectly,
    allowing attackers to compromise authentication tokens or to exploit
    implementation flaws to assume other user’s identities temporarily
    or permanently. Compromising a system’s ability to identify the
    client/user, compromises API security overall.

-   **API3:2019 Excessive Data Exposure**

    Looking forward to generic implementations, developers tend to
    expose all object properties without considering their individual
    sensitivity, relying on clients to perform the data filtering before
    displaying it to the user.

-   **API4:2019 Lack of Resources & Rate Limiting**

    Quite often, APIs do not impose any restrictions on the size or
    number of resources that can be requested by the client/user. Not
    only can this impact the API server performance, leading to Denial
    of Service (DoS), but also leaves the door open to authentication
    flaws such as brute force.

-   **API5:2019 Broken Function Level Authorization**

    Complex access control policies with different hierarchies, groups,
    and roles, and an unclear separation between administrative and
    regular functions, tend to lead to authorization flaws. By
    exploiting these issues, attackers gain access to other users’
    resources and/or administrative functions.

-   **API6:2019 Mass Assignment**

    Binding client provided data (e.g., JSON) to data models, without
    proper properties filtering based on an allowlist, usually leads to
    Mass Assignment. Either guessing objects properties, exploring other
    API endpoints, reading the documentation, or providing additional
    object properties in request payloads, allows attackers to modify
    object properties they are not supposed to. An API endpoint is
    vulnerable if it automatically converts client parameters into
    internal object properties, without considering the sensitivity and
    the exposure level of these properties. This could allow an attacker
    to update object properties that they should not have access to.

-   **API7:2019 Security Misconfiguration**

    Security misconfiguration is commonly a result of unsecure default
    configurations, incomplete or ad-hoc configurations, open cloud
    storage, misconfigured HTTP headers, unnecessary HTTP methods,
    permissive Cross-Origin resource sharing (CORS), and verbose error
    messages containing sensitive information.

-   **API8:2019 Injection**

    Injection flaws, such as SQL, NoSQL, Command Injection, etc., occur
    when untrusted data is sent to an interpreter as part of a command
    or query. The attacker’s malicious data can trick the interpreter
    into executing unintended commands or accessing data without proper
    authorization.

-   **API9:2019 Improper Assets Management**

    APIs tend to expose more endpoints than traditional web
    applications, making proper and updated documentation highly
    important. Proper hosts and deployed API versions inventory also
    play an important role to mitigate issues such as deprecated API
    versions and exposed debug endpoints.

-   **API10:2019 Insufficient Logging & Monitoring**

    Insufficient logging and monitoring, coupled with missing or
    ineffective integration with incident response, allows attackers to
    further attack systems, maintain persistence, pivot to more systems
    to tamper with, extract, or destroy data. Most breach studies
    demonstrate the time to detect a breach is over 200 days, typically
    detected by external parties rather than internal processes or
    monitoring

### Checklist 
***Note: This checklist is referenced from [31 Days of API Security Tips](https://github.com/inonshk/31-days-of-API-Security-Tips)***
-   **BFLA (Broken Function Level Authorization)**

    Leverage the predictable nature of REST to find admin API endpoints!
    E.g: you saw the following API call **GET /api/v1/users/\<id\>**
    Give it a chance and change to **DELETE / POST** to
    **create/delete**users.

    If error is received, Add:

    - [ ]   Add a "Content-length" HTTP header
    - [ ]   Change the "Content-type"

-   **BOLA (Broken Object Level Authorization)**

    - [ ]   Object and User IDS in URLs.
    - [ ]   If GUID is presented in case of IDs, implement session label
        swaping.
    - [ ]   Try Numeric IDs. If you found that an endpoint receives a
        non-numeric object ID, like GUID or an email address, give a try
        to replace it with a numeric value.

-   **Bypass Object Level Authorization**

    - [ ]   Wrap the ID with an array. Instead of `{“id”:111}` send
        `{“id”:[111]}`
    - [ ]   Wrap the ID with a JSON objectInstead of `{“id”:111}` send
        `{“id”:{“id”:111}}`
    - [ ]   Try to perform [HTTP parameter
        pollution](https://medium.com/@0xgaurang/case-study-bypassing-idor-via-parameter-pollution-78f7b3f9f59d):`*/api/get\_profile?user\_id=\<legit\_id\>&user\_id=\<victim’s\_id\>*OR*/api/get\_profile?user\_id=\<victim’s\_id\>&user\_id=\<user\_id\>*`
    - [ ]   Try to perform [JSON parameter
        pollution](http://blog.blueinfy.com/2018/07/json-parameter-pollution.html)

    `*POST
    api/get\_profile{“user\_id”:\<legit\_id\>,”user\_id”:\<victim’s\_id\>*`

    `OR *POST
    api/get\_profile{“user\_id”:\<victim’s\_id\>,”user\_id”:\<legit\_id\>}*`

-   **Security Misconfiguration**

    - [ ]   Check if unnecessary headers are implemented. (HTTP Verbose)
    - [ ]   Check if CORS policy is missing or improperly set.
    - [ ]   Check if Transport Layer Security (TLS) is missing.
    - [ ]   Check if necessary security headers are not implemented.
    - [ ]   Error messages which includes stack traces, or other sensitive
        information is exposed in error response.
    - [ ]   Check if the application discloses its full path when error is
        generated on the response.
    - [ ]   Change the JSON parameters into XML and view the error in
        response.
    - [ ]   Remote the JSON parameters and send the raw data.

-   **Injection**

    - [ ]   Test if the application is sanitizing user inputs before
        communicating it into DB.
    - [ ]   Test for SQLi payloads on the input fields or URLs as well as
        JSON parameters.
    - [ ]   Test for command injection on the input fields as well as JSON
        parameters.
    - [ ]   While uploading the files or images, change the filename into
        some SQLi payloads like 'sleep(5).jpg.
    - [ ]   Try to upload the reverse shells while uploading the files on
        the server.
    - [ ]   If the application has Export to PDF feature there is a good
        chance the developers use an external library to convert HTML
        —\> PDF behind the scenes. Try to inejct HTML elements and cause
        "Export Injection"

-   **Export Injection**

    This article will talk about a new server side vulnerability that I
    discovered in the PDF export process. Many servers are still
    vulnerable, varying from social networks to financial and
    governmental...
    
       ![](https://miro.medium.com/max/653/0*U71ekAY_IFRjxPc4.PNG)
    
-   **Broken User Authentication**

    - [ ]   Check if the API allows to perform brute force attack.
    - [ ]   Check if the API sends sensitive authentication details, such as
        auth tokens and credentials in the URL.
    - [ ]   Check if the API validates authenticity of tokens. Try sending
        request without the tokens, random tokens with same length or
        without token headers.
    - [ ]   Check if the token expires if user logs out.
    - [ ]   Check if one having low privileged user token can assess the
        resources of higher privileged user.
    - [ ]   Check for SQLi login bypasses on the login parameters.

-   **Two Factor Authenitcation Bypass Checklist**

    - [ ]   In response, if `"success:false"` change it to `"success":"true"`.
    - [ ]   If Status Code is 4xx. Try to change it to 200 OK and see if it
        bypass restriction.
    - [ ]   Check the response of the 2FA Code Triggering Request to see if
        the code is leaked.
    - [ ]   Same code can be reused.
    - [ ]   Lack of Brute-Force Protection: Possible to brute-force any
        length 2FA Code.
    - [ ]   Missing 2FA Code Integrity: The 2FA code of any user account can
        be used to bypass the victim 2FA code.
    - [ ]   Enter the code 000000 or null to bypass the 2FA protection.

-   **Excessive Data Exposure**

    - [ ]   Check if the API responds with excessive data then intended.
        -   Example: If the API is for only validating user
            registration, the API must only responds with if the user is
            registered or not, not along with the details of the user.
    - [ ]   Check if Personally Identifiable Information (PII) is leaked on
        the endpoints.
    - [ ]   Check if the API exposes other users information if it takes the
        id,name parameters for fetching the informations despite of
        token.

-   **Lack of resources and Rate Limiting**
    - [ ]   Check whether it has brute force protection of not.
    - [ ]   Check if the user can perform the same request for multiple long
        times. A huge number of such request can disrupt the server.
    - [ ]   Try injecting a large payloads on the input parameters and
        analyze the response.
    - [ ]   Check the execution timeouts is either too high or too low.
    - [ ]   Try to upload very large file in upload field.
    - [ ]   Try inserting large numbers in number of records per page to
        return in a single request response.
        -   Example: Found a limit/page param? (/api/news?limit=100). It
            might be vulnerable to Layer 7 DoS. Try to send a long value
            (e.g: limit=9999999999999) and see what happens.

-   **Mass Assignment**
    - [ ]   Check if the API discloses additional object properties when the
        request is converted into GET. It does not need to be converted
        on GET always.

    - [ ]   If it discloses, send the request with PUT method.

        -   Example-1: If the request is for login /api/v2/user/ and the
            parameters are username and password.
        -   If the response body contains admin=false object.
        -   Attack can again send the PUT request method with parameter
            username, password and admin=true on above endpoint to
            update itself into the admin.

        ![](https://pbs.twimg.com/media/ENpsW25XYAAjEJE?format=jpg&name=medium)

        -   Example -2 : A ride sharing application provides a user the
            option to edit basic information for their profile. During
            this process, and API call is sent to PUT /ap/v1/users/me
            with the following legitimate JSON object:
        ``` {#e77a8802-a10d-448b-8ac1-3d469f41d033 .code}
        {"user_name":"inons","age":24}
        ```
        -   The request GET /api/v1/users/me includes an additional
            credit\_balance property:
        ``` {#78607019-bac4-4fd9-a3b8-0d2cec501d06 .code}
        {"user_name":"inons","age":24,"credit_balance":10}.
        ```
        -   The attacker replays the first request with the following
            payload:
        ``` {#c7e72ad1-1cb4-49e8-872d-dd57bed16ccb .code}
        {"user_name":"attacker","age":60,"credit_balance":99999}
        ```
        -   Since the endpoint is vulnerable to mass assignment, the
            attacker receives credits without paying.

-   **Improper Assets Management**

    - [ ]   Check for the different outdated versions of the API. They can
        still contains old vulnerabilities which are unpatched.
        -   Example: `/api/v2/user`, replace it with `/api/v1/user` and
            analyze the response.


## References
- [31 Days of API Security Tips](https://github.com/inonshk/31-days-of-API-Security-Tips)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Inonst Export Injection](https://inonst.medium.com/export-injection-2eebc4f17117)
