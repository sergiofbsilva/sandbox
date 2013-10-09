#OAuth Authorization Workflow

This guide will show you how to authorize your web application using OAuth2 and how to use the Fénix API so you can access Fénix users' information with their authorization.

##Table of Contents

- [1. Register Application](#1-register-application)
- [2. Request User Permission](#2-request-user-permission)
	- [Error Handling](#error-handling)
- [3. Request the Access Token](#2-request-the-access-token)
	- [Error Handling](#error-handling-1)
- [4. Make an API request](#3-make-an-api-request)
	- [Error Handling](#error-handling-2)
- [5. Refreshing the access token](#3-refreshing-the-access-token)
	- [Error Handling](#error-handling-3)

#1. Register Application

This section will explain how to register your application in Fénix.

1. Login at Fénix and select `Personal` -> `Manage Applications`.
    If it is not present, please contact support so you can get the role `DEVELOPER`.

2. Select `Create Application`
3. Fill the requested fields
    * Logo - should be an image with 64x64 pixels
    * Name - Application Name
    * Description - Short text about your application
    * Site URL - The application site with information about your application (optional, not to be confused with Redirect URL)
    * Scopes - The scopes your application needs to request the users authorization
    * Redirect URL - The endpoint in your application to proceed the OAuth2 flow.

After completing this registration process, you are ready to start the authorization workflow.


#2. Request User Permission

This section will explain how you can request the user permission so your application can access the Fénix API.

1.  Your application must redirect the user to the following URL
        
        https://fenix.ist.utl.pt/oauth/userdialog?client_id=<client_id>&redirect_uri=<redirect_uri>
    * You should get the `client_id` and `redirect_uri` values from the application details in Fénix.

The user will be redirected to a web page where it will be requested to authorize your application to access the scopes you selected in the application creation process. It will request user login if necessary.

##Error Handling
If the user denies the authorization request the user will be redirected to

    <redirect_uri>?error=access_denied&error_description=User didn't allow the application

At this point you can't continue the authorization workflow.

#2. Request the Access Token

If the user authorizes your application, the web browser will be redirected to the `redirect_uri` like this
    
    <redirect_uri>?code=XXXXXXXXXXX

At this point you must issue a `POST` request on
    
    https://fenix.ist.utl.pt/oauth/access_token?client_id=<client_id>&client_secret=<client_secret>&redirect_uri=<redirect_uri>&code=<code>&grant_type=authorization_code

* The `client_id`, `redirect_uri`, `client_secret` can be found the on the application details in Fénix.
* The `code` is passed as a parameter to `redirect_uri` where the current request must be made.

If everything goes smoothly, you should get an `application/json` 200 OK response with the following format
```json
{"access_token":"IGNhbiBjb252ZXJ0IHRleHRzIHVzaW5nIHNldmVyYWwgY29kZSBwYWdlcyAodXNpbmcgQ2hhclNl", "refresh_token":"dCBwcm9wZXJ0eSkgZnJvbSBVbmljb2RlIHN0cmluZyB0byBieXRlIGFycmF5IGFuZCB0aGVuIGNv", expires:"3600"}
```

##Error Handling

If something goes wrong  a `application/json` `400 BAD_REQUEST` response will be issued in this format:
```json
{"error":"invalid_grant","error_description":"..."}
```

The following descriptions can occur:

* `Client ID not recognized` - when the client id is not found (please check the application details)
* `Credentials or redirect_uri don't match` - When the `client_secret` is wrong or the `redirect_uri` is not the same that is registered
* `Code Invalid`- the `code` is no longer valid
* `Code expired` - the `code` has expired

#3. Make an API request

At this point you already have an `access_token` and you want to invoke the Fénix API on behalf of the user who previously gave your application permissions.

The API documentation is described [here]()

The API base URL is `https://fenix.ist.utl.pt/api/v1`

Each endpoint belongs to a scope except the endpoints that are public.
When requesting a scoped endpoint the query param `access_token` must be present in the URL. For example:

    https://fenix.ist.utl.pt/api/v1/person?access_token=<access_token>

##Error Handling

If something goes wrong  a `application/json` `HTTP 401 Not Authorized` response will be issued in this forma
t:
```json
{"error":"...","error_description":"..."}
```

The `error` can be one of the following

* `invalidScope` - Application doesn't have permissions to this endpoint.
* `accessTokenInvalid` - Access Token doesn't match.
* `accessTokenExpired` - The access has expired. Please use the refresh token endpoint to generate a new one.
* `accessTokenInvalidFormat` - Access Token not recognized.


#3. Refreshing the access token

If the `access_token` expires, the `refresh_token` must used to get a new one. The `refresh_token` never expires.

To get a new `access_token` you must do a `POST` request on :

    https://fenix.ist.utl.pt/oauth/refresh_token?client_id=<client_id>&client_secret=<client_secret>&refresh_token=<refresh_token>

If everything goes smoothly, you should get an `application/json` 200 OK response with the following format
```json
{"access_token":"IGNhbiBjb252ZXJ0IHRleHRzIHVzaW5nIHNldmVyYWwgY29kZSBwYWdlcyAodXNpbmcgQ2hhclNl", "expires":"3600"}
```

##Error Handling
If something goes wrong  a `application/json` `HTTP 401 Not Authorized` response will be issued in this forma
t:
```json    
{"error":"...","error_description":"..."}
```
The `error` can be one of the following

* `invalid_grant` - Credentials or redirect_uri don't match
* `refreshTokenInvalid` - Refresh token doesn't match
* `refreshTokenInvalidFormat` - Refresh Token not recognized.
