
> [!NOTE]
>  ↓↓⚡ Before Exploiting the OAuth - READ This ↓↓⚡

# 🔍Identifying OAuth Usage in an Application

![lens showing how to detect an OAuth framework | 500](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1724169604593.png)

The first indication that an application uses OAuth is often found in the login process. Look for options allowing users to log in using external service providers like Google, Facebook, and GitHub. These options typically redirect users to the service provider's authorization page, which strongly signals that OAuth is in use.

## Detecting OAuth Implementation

When analyzing the network traffic during the login process, pay attention to HTTP redirects. OAuth implementations will generally redirect the browser to an authorization server's URL. This URL often contains specific query parameters, such as `response_type`, `client_id`, `redirect_uri`, `scope`, and `state`. These parameters are indicative of an OAuth flow in progress. For example, a URL might look like this:  

```
https://dev.coffee.thm/authorize?response_type=code&client_id=AppClientID&redirect_uri=https://dev.coffee.thm/callback&scope=profile&state=xyzSecure123

```
## Identifying the OAuth Framework

Once you have confirmed that OAuth is being used, the next step is to identify the specific framework or library the application employs. This can provide insights into potential vulnerabilities and the appropriate security assessments.

> [!NOTE] Here are some strategies to identify the OAuth framework:
> 1. **HTTP Headers and Responses**: Inspect HTTP headers and response bodies for unique identifiers or comments referencing specific OAuth libraries or frameworks.
> 2. **Source Code Analysis**: If you can access the application's source code, search for specific keywords and import statements that can reveal the framework in use. For instance, libraries like `django-oauth-toolkit`, `oauthlib`, `spring-security-oauth`, or `passport` in `Node.js`, each have unique characteristics and naming conventions.
> 3. **Authorization and Token Endpoints**: Analyze the endpoints used to obtain authorization codes and access tokens. Different OAuth implementations might have unique endpoint patterns or structures. For example, the `Django OAuth Toolkit` typically follows the pattern `/oauth/authorize/` and `/oauth/token/`, while other frameworks might use different paths.
> 4. **Error Messages**: Custom error messages and debug output can inadvertently reveal the underlying technology stack. Detailed error messages might include references to specific OAuth libraries or frameworks.


---
# 💥Exploiting OAuth - Stealing OAuth Token


Tokens play a critical role in the OAuth 2.0 framework, acting as digital keys that grant access to protected resources. These tokens are issued by the authorization server and redirected to the client application based on the `redirect_uri` parameter. This redirection is crucial in the OAuth flow, ensuring that tokens are securely transmitted to the intended recipient. However, if the `redirect_uri` is not well protected, attackers can exploit it to hijack tokens.  

### ↳ Role of Redirect_URI

The `redirect_uri` parameter is specified during the OAuth flow to direct where the authorization server should send the token after authorization. This URI must be pre-registered in the application settings to prevent open redirect vulnerabilities. During the OAuth process, the server checks that the provided `redirect_uri` matches one of the registered URIs.

### ↳ Vulnerability

An insecure `redirect_uri` can lead to severe security issues. If attackers gain control over any domain or URI listed in the `redirect_uri`, they can manipulate the flow to intercept tokens. Here’s how this can be exploited:

- Consider an OAuth application with the following registered redirect URIs as shown below:

![coffeeshopapp application panel showing redirect_uri](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1723542692246.png)  

- **Attacker's Strategy**: If an attacker gains control over `dev.bistro.thm`, they can exploit the OAuth flow. By setting the `redirect_uri` to `http://dev.bistro.thm/callback`, the authorization server will send the token to this controlled domain.
- **Crafted Attack:** The attacker initiates an OAuth flow and ensures the `redirect_uri` points to their controlled domain. After the user authorizes the application, the token is sent to `http://dev.bistro.thm/callback`. The attacker can now capture this token and use it to access protected resources.


> [!NOTE]
> > IN OAuth request change `redirect_uri` path with Burp-Suite


---
# 💥Exploiting OAuth - CSRF in OAuth


The **state** parameter in the OAuth 2.0 framework protects against CSRF attacks, which occur when an attacker tricks a user into executing unwanted actions on a web application where they are currently authenticated. In the context of OAuth, CSRF attacks can lead to unauthorized access to sensitive resources by hijacking the OAuth flow. The state parameter helps mitigate this risk by maintaining the integrity of the authorization process.

### ↳Vulnerability of Weak or Missing State Parameter

The state parameter is an arbitrary string that the client application includes in the authorization request. When the authorization server redirects the user back to the client application with the authorization code, it also includes the state parameter. The client application then verifies that the state parameter in the response matches the one it initially sent. This validation ensures that the response is not a result of a CSRF attack but a legitimate continuation of the OAuth flow.  

For instance, consider an OAuth implementation where the state parameter is either **missing** or **predictable** (e.g., a static value like "state" or a simple sequential number). An attacker can initiate an OAuth flow and provide their malicious redirect URI. After the user authenticates and authorizes the application, the authorization server redirects the authorization code to the attacker's controlled URI, as specified by the weak or absent state parameter.

> [!NOTE]
> To perform exploit the vulnerability. Attacker simply replace the OAUTH `code` with the victim code which you get from Authorization server . If  this successful you can see you get the access to the system.

---
# 💥 Exploiting OAuth - Implicit Grant Flow




---
# 💥 Other Vulnerabilities and Evolution of                  OAuth 2.1


