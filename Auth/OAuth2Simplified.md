# OAuth 2.0 Simplified

## 1. Getting Ready

* In this chapter we will walk through the things you need to know when you’re building an app that talks to an existing
  OAuth 2.0 API.

### 1-1.Creating an Application

* Get the client_id (and a client_secret in some cases) from the resource service that you’ll use when your app
  interacts with the service.
* Register one or more redirect URLs the application will use.

### 1-2. Redirect URLs and State

* Redirect URLs
   * OAuth 2.0 APIs will only redirect users to a URL that was previously registered with the service, in order to
     prevent redirection attacks where an authorization code or access token can be intercepted by an attacker.
   * OAuth services should be looking for an exact match of the redirect URL.
* State Parameter
   * The “state” parameter can be used to encode application state, but it must also include some amount of random data
     if you’re not also including PKCE parameters in the request.
   * It’s important to generate a “state” parameter to use to protect the client from CSRF attacks. This is a random
     string that the client generates and stores in the session.
   * Whatever state value you pass in during the initial authorization request will be returned after the user
     authorizes the application.
   * So we can verify it matches the state stored in the session before exchanging the authorization code for an access
     token.
   * You could for example encode a redirect URL in something like a JWT, and parse this after the user is redirected
     back to your application, so you can take the user back to the appropriate location after they sign in.
   * Note that unless you are using a signed or encrypted method like JWT to encode the state parameter, you should
     treat it as untrusted/unvalidated data when it arrives at your redirect URL, since it’s trivial for anyone to
     modify that parameter on the redirect back to your app.

## 2. Accessing Data in an OAuth Server

*
   * In this chapter we will walk through how to access your data at an existing OAuth 2.0 server.

### 2-1. Obtaining an Access Token

* Note: We would check for errors returned from auth service and display an appropriate message to the user.

## 4. Server-Side Apps

* Server-side apps use the authorization_code grant type. In this flow, after the user authorizes the application, the
  application receives an “authorization code” which it can then exchange for an access token.
*

## 5. Single-Page Apps

* Since the entire source is available to the browser, they cannot maintain the confidentiality of a client secret.
* The best option is to use the PKCE extension to protect the authorization code in the redirect.

## 17. Protecting Apps with PKCE

* Proof Key for Code Exchange (abbreviated PKCE, pronounced “pixie”) is an extension to the authorization code flow to
  prevent CSRF and authorization code injection attacks.
* The client first creating a secret on each authorization request, and then using that secret again when exchanging the
  authorization code for an access token.
* PKCE was originally designed to protect the authorization code flow in mobile apps, and was later recommended to be
  used by single-page apps as well. In later years, it was recognized that its ability to prevent authorization code
  injection makes it useful for every type of OAuth client, even apps running on a web server that use a client secret.

### 17-1. Authorization Request








