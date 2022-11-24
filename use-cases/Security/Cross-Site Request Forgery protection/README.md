Cross-Site Request Forgery protection
=====================================

* 1 [Presentation](#presentation)
* 2 [Backup](#backup)
* 3 [Context](#context)
* 4 [Attack vectors](#attack-vectors)
* 5 [How to prevent CSRF attacks?](#how-to-prevent-csrf-attacks)
    * 5.1 [Tokens](#tokens)
    * 5.2 [Verify Referer and Origin headers](#verify-referer-and-origin-headers)
    * 5.3 [Set-Cookie security attributes](#set-cookie-security-attributes)
* 6 [Limitations](#limitations)
* 7 [Solution description](#solution-description)
    * 7.1 [SWF - CSRF Protection](#swf-csrf-protection)
    * 7.2 [Recommendations for complete protection against CSRF](#recommendations-for-complete-protection-against-csrf)


Presentation
------------

Cross-Site Request Forgery (abbreviated CSRF – pronounced “sea-surf” – or sometimes XSRF) is a cyber-attack where a hacker forces a legitimate user to perform an unwanted action on a trusted website in which the user is already authenticated. They unknowingly become an accomplice. Since the attack is being set in motion by the legitimate user on his own, it is capable of circumventing many authentication systems.

A CSRF attack is an effective attack because, generally, browser requests involve all the cookies, including the session cookies. Therefore, if the user is already authenticated to the site, the latter cannot distinguish between legitimate requests and forged ones.

The criticality and risks of CSRF attacks depend on the actions that can be made via these attacks. Hackers often attempt to get users to perform administrative tasks against their will.

For more details, see:

*   [https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site\_Request\_Forgery\_Prevention\_Cheat\_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
*   [https://owasp.org/www-community/attacks/csrf](https://owasp.org/www-community/attacks/csrf)

Backup
------

Sub-Workflow can be downloaded here: [SWF - CSRF Protection.backup](./backup/SWF%20-%20CSRF%20Protection.backup).

The Sub-Workflow is available by default since the 6.7.0 version.

Context
-------

The hacker’s goal here is to trigger an action within the victim’s operating context, using their rights and authorizations. To accomplish this, the user is forced to send a form to the web application or to visit an URL that is capable of setting off a particular action in a totally invisible way. Since the victim is already authenticated on the site when the action is performed, no request for authentication is issued.

Attack vectors
--------------

CSRF may exploit these vectors:

*   GET method: this is typically the result of poor web application development, allowing a simple HTML link to trigger an unwanted action.
*   POST method: this is more aggressive and is the most often encountered. They simulate the sending of a form, and thus of a more complex and sensitive operation.
*   PUT and DELETE methods: as many applications make API calls using methods like PUT and DELETE, these can be used but will require specific requirements, like a cross-origin resource sharing enabled on the web application (`Access-Control-Allow-Origin: *`).

How to prevent CSRF attacks?
----------------------------

### Tokens

The main solution lies in the development of the application. It consists of implementing tokens for sensitive operations in order to ensure that when the application receives an action from the user, the action is really derived from the last page visited by that user. Since the token is generated randomly, a hacker won’t be able to find it and include it in the request he wants to send to the site via the victim.

### Verify _Referer_ and _Origin_ headers

A secondary solution is to pay attention to _Referer_ and _Origin_ headers. These HTTP headers, included in POST requests, identify – respectively – the page and the domain where the current request originated. Therefore we can verify them and determine whether a POST request containing variables (typically the result of sending a form) is indeed derived from our web application and not from a third source set up by an attacker.

This solution can be implemented on the WAF, since it is not directly dependent on the web application’s code and structure (see next parts).

### Set-Cookie security attributes

To add more security against CSRF attacks, you can add **SWF - Secure cookies** sub-workflow to your workflows to secure cookies sent by server.
More information about it and the backup file can be found here: [Secure Cookies](../Secure%20Cookies)

Limitations
-----------

Here, we chose providing a protection based on the _Referer_ and _Origin_ headers verification, with the usage of the security attribute on Set-Cookies.

The solution can protect against attacks derived from POST requests. However, attacks derived from GET requests cannot be stopped without risking false positives, especially on the first request (possibly no _Referer_ header yet).

Solution description
--------------------------

We currently provide 2 Sub-Workflows to mitigate CSRF attacks: "SWF - CSRF protection" and "SWF - Add SameSite in Set-Cookies"

The first "SWF - CRSF Protection" has to be placed after the start node and before security engines (see figure below).

![](./attachments/csrf-workflow.png)

### SWF - CSRF Protection

This node must be configured by entering the domain name that the headers _Referer_ and _Origin_ will have to respect:

![](./attachments/swf-csrf-protection.png)

*   There is no need to enter the '_https://_' scheme before, nor the final '_/_'. If the application uses a custom port (not 80 or 443), the port has to be added in the domain name parameter. The SWF will check HTTP and HTTPS at the same time. For example: 'www.app.com', 'www.app.com:8443';
*   The security mode allows to block and log, or only log, requests that do not match the domain name;
*   There is the possibility to enable checks for one of the two headers, both of them, and for the specified methods;
*   GET method check is disabled by default to avoid the risk of false positives. Enable checks on GET method with a log only security mode will allow checking for false positives without blocking.

#### How does this Workflow node operate?

The SWF will first check the method of the request:

*   If POST/PUT requests: it verifies that the received request is really a POST or PUT request and contains the body or query parameters. If so, it will check the validity of the _Referer_ and _Origin_ headers.
*   If GET/DELETE: it verifies that the received request is really a GET or DELETE request and contains query parameters (for GET). If so, it will check the validity of the _Referer_ header

**GET requests**

Verification on GET methods is not enabled by default to avoid false positives. Indeed, the first request arrives most of the time without the _Referer_ header. If the first requests are without any query parameters, the SWF will not check the _Referer_ header.

The SWF has also other security modes:

*   Block and log when not matching or not present (strict).
*   Block and log when not matching (lax): default mode. Requests will not be blocked when headers are not in the request.
*   Log only when not matching (log).

### Recommendations for complete protection against CSRF

*   Ask for confirmations from the user for critical actions, at the risk of weighing down the progression of the forms.
*   Avoid using HTTP GET method to perform actions or sent data. This technique will naturally eliminate simple attacks based on images but will let attacks using JavaScript, since they are capable of launching HTTP POST requests very easily.
*   Use validity tokens in forms. Ensure that a posted form is accepted only if it has been produced a few minutes earlier; the validity token serves as proof. Hence it must be sent as a parameter and verified on the server-side.
*   Verify the Referer in sensitive pages.
*   Implement the UBIKA WAAP "SWF - CSRF Protection".
*   Add the [SWF - Secure Cookies](../Secure%20Cookies) to forbid cookies to be sent along cross-site requests would help to mitigation some CSRF attacks by avoiding the authentication when going on the third party (if authentication is linked).
