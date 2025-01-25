This task will discuss the key concepts for understanding OAuth, specifically OAuth 2.0. These concepts form the foundation for understanding how the OAuth 2.0 framework was built. As a pentester or a secure coder, it is essential to understand these concepts to pentest a website or write code without a vulnerability. To make these concepts more relatable, we will explain them through a daily routine example: using a coffee shop's mobile app to order and pay for coffee.![splashing coffee on oauth icon |300](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1724169244381.png)

## **Resource Owner**

The resource owner is the **person** or **system** that controls certain data and can authorize an application to access that data on their behalf. This concept is fundamental as it centres around user consent and control. For example, you are the resource owner as a coffee shop customer. You can control your account information and grant the coffee shop's mobile app permission to access your data.  

## **Client**  

The client can be a **mobile app** or a **server-side web application**. It acts as an intermediary, requesting access to resources and performing actions as permitted by the resource owner. For example, the coffee shop's web app, which you use to order and pay for coffee, is the client. Your authorization is needed to access your account details and payment information.

## **Authorization Server**

The authorization server is responsible for **issuing access tokens to the client** after successfully authenticating the resource owner and obtaining their authorization. The authorization server plays a crucial role in the OAuth process by ensuring the client is granted permission only after legitimate user authentication and consent. For example, the coffee shop's backend system that handles authentication and authorization is the authorization server. It verifies your credentials and grants the web app permission to access your account.  

## **Resource Server**

The server hosting the protected resources can **accept and respond** **to protected resource requests** using access tokens. This server ensures that only authenticated and authorized clients can access or manipulate the resource owner's data. For example, the resource server is the coffee shop's database that stores your account information, order history, and payment details. It responds to requests from the web app, allowing it to retrieve and modify your data.

## **Authorization Grant**

The client uses a credential representing the resource owner's authorization (to access their protected resources) to obtain an access token. The primary grant types are `Authorization Code`, `Implicit`, `Resource Owner Password Credentials`, and `Client Credentials`. For example, when you first log in to the coffee shop's app, you are given an authorization grant (like entering your username and password). The app uses this grant to get an access token from the authorization server. We will discuss it in detail in the next task.

## **Access Token**

A credential that the client can use to access protected resources on behalf of the resource owner. It has a limited lifespan and scope. Access tokens are essential for maintaining secure and protected communication between the client and resource server without repeatedly asking the resource owner for credentials. For example, once you log in to the coffee shop's app, it receives an access token, which allows the app to access your account to place orders and make payments without asking you to log in again for a specific period. ![key icon depicting access token | 200](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1724169379373.png)

## **Refresh Token**

A credential that the client can use to obtain a new access token without requiring the resource owner to re-authenticate. Refresh tokens are typically long-lived and provide a way to maintain user sessions without frequent login interruptions. For example, when your access token expires, the web app will use a refresh token to get a new access token, so you don’t have to log in again.

## **Redirect URI**

The URI to which the authorization server will redirect the resource owner’s user-agent after the grant or denial of the authorization. It checks if the client for which the authorization response has been requested is correct. For instance, after interacting with the coffee shop app and logging in, you will be redirected to the authorization server by the app page in the coffee shop’s app, commonly known as the redirect URI, to confirm that you successfully logged in.

## **Scope**

Scopes are a mechanism for limiting an application's access to a user's account. They allow the client to specify the level of access needed and the authorization server to inform the user what access levels the application is requesting. Scopes help enforce the `principle of least privilege`. For example, the coffee shop's app may request different scopes, such as access to your order history and payment details. As the resource owner, you can see what information the app requests access to and grant or deny permissions.  

## **State Parameter**  

An **optional** parameter maintains the state between the client and the authorization server. It can help prevent CSRF attacks by ensuring the response matches the client's request. The state parameter is a crucial part of securing the OAuth flow. For example, when you initiate the login process, the coffee shop's app sends a state parameter to the authorization server. This parameter helps ensure that the response you receive is linked to your original request, protecting against certain types of attacks.

## **Token & Authorization Endpoint**

The authorization server's endpoint is where the client exchanges the authorization grant (or refresh token) for an access token. In contrast, the authorization endpoint is where the resource owner is authenticated and authorizes the client to access the protected resources.

By familiarizing yourself with the topics, you will easily understand the exploitation techniques and associated vulnerabilities in the upcoming tasks.



> [!Next] NEXT ⇒
>[[OAuth Grant Types]]

 