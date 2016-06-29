# Android Native Application with AppAuth
Sample application for communicating with OAuth 2.0 and OpenID Connect providers. Demonstrates single-sign-on (SSO) with [AppAuth for Android](https://github.com/openid/AppAuth-Android).

## Installation
This sample application was tested with an Okta org. If you do not have an Okta org, you can easily [sign up for a free Developer Okta org](https://www.okta.com/developer/signup/).

### Okta Application:
To properly set up an application that supports OAuth 2.0 and OpenID Connect, follow these steps.
  - Application Type
    - Native
  - Allowed Grant Types
    - Authorization Code, Refresh Token
  - Redirect URI:
    - Reverse DNS with route (*Ex: com.otkapreview.jordandemo:/openid*)
  - Client Credientials
    - Proof Key for Code Exchange

### Sample Application
Once the project is cloned, simply open the project in [Android Studio](https://developer.android.com/studio/index.html) and `Import` the project.

Update the following in your `OktaConfiguration.java` file:
  - kIssuer
    - Your okta/oktapreview domain (*Ex: https://jordandemo.oktapreview.com*)
  - kClientID
   	- Okta Application ID
  - kRedirectURI
    - Reverse DNS w/ route (*Ex: com.oktapreview.jordandemo:/openid*)
