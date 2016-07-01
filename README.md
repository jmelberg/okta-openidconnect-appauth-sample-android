# Android Native Application with AppAuth
Sample application for communicating with OAuth 2.0 and OpenID Connect providers. Demonstrates single-sign-on (SSO) with [AppAuth for Android](https://github.com/openid/AppAuth-Android).

## Running the Sample with your Okta Organization

###Pre-requisites
This sample application was tested with an Okta org. If you do not have an Okta org, you can easily [sign up for a free Developer Okta org](https://www.okta.com/developer/signup/).

1. Verify OpenID Connect is enabled for your Okta organization. `Admin -> Applications -> Add Application -> Create New App -> OpenID Connect`
  - If you do not see this option, email [developers@okta.com](mailto:developers@okta.com) to enable it.
2. In the **Create A New Application Integration** screen, click the **Platform** dropdown and select **Native app only**
3. Press **Create**. When the page appears, enter an **Application Name**. Press **Next**.
4. Add the reverse DNS notation of your organization to the *Redirect URIs*, followed by a custom route. *(Ex: "com.oktapreview.jordandemo:/oidc")*
5. Click **Finish** to redirect back to the *General Settings* of your application.
6. Select the **Edit** button to configure the **Allowed Grant Types** and **Client Authentication**
  - Ensure *Authorization Code* and *Refresh Token* are selected in **Allowed Grant Types**
  - Verify *Proof Key for Code Exchange (PKCE)* is the default **Client Authentication**
7. **Save** the application.
8. Copy the **Client ID**, as it will be needed for the `OktaConfiguration.java` configuration file.
9. Finally, select the **People** tab and **Assign to People** in your organization.

### Configure the Sample Application
Once the project is cloned, simply open the project in [Android Studio](https://developer.android.com/studio/index.html) and `Import` the project.

Update the **kIssuer**, **kClientID**, and **kRedirectURI** in your `OktaConfiguration.java` file:
```java
public class OktaConfiguration {
    public String kIssuer = "https://example.com";
    public String kClientID = "CLIENT_ID";
    public String kRedirectURI = "com.example:/openid";
    public String kAppAuthExampleAuthStateKey = "com.okta.openid.authState";
    public String apiEndpoint = "https://example.server.com";

    public OktaConfiguration() {}

}
```

## Running the Sample Application

| Get Tokens      | Get User Info  | Refresh Token  | Revoke Token   | Call API       | Clear Tokens   |
| :-------------: |:-------------: |:-------------: |:-------------: |:-------------: |:-------------: |
| ![Get Tokens](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/key_circle.imageset/key.png)| ![Get User Info](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/Reporting.imageset/Reporting.png)| ![Refresh Token](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/refresh.imageset/api_call.png)| ![Revoke Token](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/revoke.imageset/revoke.png) | ![Call API](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/refresh.imageset/api_call.png) | ![Clear Tokens](https://raw.githubusercontent.com/jmelberg/okta-openidconnect-appauth-sample-swift/master/OpenIDConnectSwift/Assets.xcassets/ic_key.imageset/MFA_for_Your_Apps.png)|

###Get Tokens
Interacts with the Authorization Server by using the discovered values from the domain's `.well-known/openid-configuration`. If the endpoints are found, the method `makeAuthRequest` generates the request by passing in the required scopes and opening up an in-app browser to get the User credentials.

```java
  private void makeAuthRequest(@NonNull AuthorizationServiceConfiguration authorizationServiceConfiguration) {
    AuthorizationRequest authorizationRequest = new AuthorizationRequest.Builder(
      authorizationServiceConfiguration,
      configuration.kClientID,
      AuthorizationRequest.RESPONSE_TYPE_CODE,
      Uri.parse(configuration.kRedirectURI)).setScope(SCOPE).build();
    Log.d(TAG, "Making auth request to " + authorizationServiceConfiguration.authorizationEndpoint);
    mAuthService.performAuthorizationRequest(
      authorizationRequest,
      createPostAuthorizationIntent(
        this.getApplicationContext(),
        authorizationRequest,
        authorizationServiceConfiguration.discoveryDoc
      ));
  }
```
###Get User Info
If the user is authenticated, fresh tokens are generated for calling the `/userinfo` endpoint to retrieve user data. If received, the output is printed to the console and a UIAlert.

###Refresh Tokens
The AppAuth method `performTokenRequst` is used to refresh the current **access token** if the user is authenticated.

###Revoke Tokens
If authenticated, the token is passed to the `/revoke` endpoint to be revoked, and sets the `setNeedsTokenRefresh` method to `true`.

```java
public void callRevokeEndpoint() {
  try {
    if (mAuthState.getAuthorizationServiceConfiguration() == null) {
      Log.d(TAG, "Cannot make request without service configuration");
    }

    AuthorizationServiceDiscovery discoveryDoc = getDiscoveryDocFromIntent(getIntent());
    if (discoveryDoc == null) { throw new IllegalStateException("no available discovery doc"); }

    try {
      URL url = new URL(discoveryDoc.getIssuer() + "/oauth2/v1/revoke");
      HttpURLConnection conn = (HttpURLConnection) url.openConnection();
      String params = "client_id="+configuration.kClientID+"&token="+mAuthState.getAccessToken();
      byte[] postParams = params.getBytes("UTF-8");
      conn.setRequestMethod("POST");
      ...
      String response = readStream(conn.getInputStream());
      if (conn.getResponseCode() == 200 || conn.getResponseCode() == 204) {
        createAlert("Token Revoked", "Previous access token is considered invalid");
        Log.d(TAG, "Previous access token is considered invalid");
        mAuthState.setNeedsTokenRefresh(true);
      } else {
        Log.e(TAG, "Unable to revoke access token");
      }
      ...
    }
```

###Call API
Passes the current access token *(active or inactive)* to a resource server for validation. Returns an api-specific details about the authenticated user.

###Clear Tokens
Sets the current `authState` to `null`.
