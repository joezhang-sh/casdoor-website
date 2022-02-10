---
title: Casdoor SDK
---

## Introduction

Compared to the standard OIDC protocol, Casdoor provides more functionalities in its SDK, like user management, resource uploading, etc. Connecting to Casdoor via Casdoor SDK costs more time than using a standard OIDC client library but will provide the best flexibility and the most powerful API.

Casdoor SDKs can be divided into two categories:

1. **Frontend SDKs**: Javascript SDK for websites, Android or iOS SDKs for Apps. Casdoor supports providing authentication for both websites and mobile Apps.
2. **Backend SDK**: SDKs for backend languages like Go, Java, Node.js, Python, PHP, etc.

:::tip
If your website is developed in a frontend and backend separated manner, then you can use the Javascript SDK: `casdoor-js-sdk` to integrate Casdoor in frontend. If your web application is a traditional website developed by JSP or PHP, you can just use the backend SDKs only.
:::

| Casdoor Frontend SDK | Description      | Source code                                    |
|----------------------|------------------|------------------------------------------------|
| Javascript SDK       | For websites     | https://github.com/casdoor/casdoor-js-sdk      |
| Android SDK          | For Android apps | https://github.com/casdoor/casdoor-android-sdk |
| iOS SDK              | For iOS apps     | https://github.com/casdoor/casdoor-ios-sdk     |

Next, use one of the following backend SDKs based on the language of your backend:

| Casdoor Backend SDK | Source code                                   |
|---------------------|-----------------------------------------------|
| Go SDK              | https://github.com/casdoor/casdoor-go-sdk     |
| Java SDK            | https://github.com/casdoor/casdoor-java-sdk   |
| Node.js SDK         | https://github.com/casdoor/casdoor-nodejs-sdk |
| Python SDK          | https://github.com/casdoor/casdoor-python-sdk |
| PHP SDK             | https://github.com/casdoor/casdoor-php-sdk    |
| .NET SDK            | https://github.com/casdoor/casdoor-dotnet-sdk |

For a full list of the official Casdoor SDKs, please see: https://github.com/casdoor?q=sdk&type=all&language=&sort=

## How to use Casdoor SDK?

### 1. Backend SDK configuration

When your application starts up, you need to initialize the Casdoor SDK config by calling the `InitConfig()` function with required parameters. Take casdoor-go-sdk as example: https://github.com/casbin/casnode/blob/6d4c55f5c9a3c4bd8c85f2493abad3553b9c7ac0/controllers/account.go#L51-L64

```go
var CasdoorEndpoint = "https://door.casbin.com"
var ClientId = "541738959670d221d59d"
var ClientSecret = "66863369a64a5863827cf949bab70ed560ba24bf"
var CasdoorOrganization = "casbin"
var CasdoorApplication = "app-casnode"

//go:embed token_jwt_key.pem
var JwtPublicKey string

func init() {
	auth.InitConfig(CasdoorEndpoint, ClientId, ClientSecret, JwtPublicKey, CasdoorOrganization, CasdoorApplication)
}
```

All the parameters for `InitConfig()` are explained as follows:

| Parameter        | Must | Description                                                                   |
|------------------|------|-------------------------------------------------------------------------------|
| endpoint         | Yes  | Casdoor Server URL, like `https://door.casbin.com` or `http://localhost:8000` |
| clientId         | Yes  | Client ID for the Casdoor application                                         |
| clientSecret     | Yes  | Client secret for the Casdoor application                                     |
| jwtPublicKey     | Yes  | The public key for the Casdoor application's cert                             |
| organizationName | Yes  | The name for the Casdoor organization                                         |
| applicationName  | No   | The name for the Casdoor application                                          |

### 2. Frontend configuration

First, install `casdoor-js-sdk` via NPM or Yarn:

```json
npm install casdoor-js-sdk
```

Or:

```json
yarn add casdoor-js-sdk
```

Then define the following utility functions (better in a global JS file like `Setting.js`):

```js
import Sdk from "casdoor-js-sdk";

export function initCasdoorSdk(config) {
   CasdoorSdk = new Sdk(config);
}

export function getSignupUrl() {
   return CasdoorSdk.getSignupUrl();
}

export function getSigninUrl() {
   return CasdoorSdk.getSigninUrl();
}

export function getUserProfileUrl(userName, account) {
   return CasdoorSdk.getUserProfileUrl(userName, account);
}

export function getMyProfileUrl(account) {
   return CasdoorSdk.getMyProfileUrl(account);
}

export function getMyResourcesUrl(account) {
   return CasdoorSdk.getMyProfileUrl(account).replace("/account?", "/resources?");
}

export function signin() {
   return CasdoorSdk.signin(ServerUrl);
}
```

In the entrance file of your frontend code (like `index.js` or `app.js` in React), you need to initialize the `casdoor-js-sdk` by calling the `InitConfig()` function with required parameters. The first 4 parameters should use the same value as the Casdoor backend SDK. The last parameter `redirectPath` is relative path for the redirected URL, returned from Casdoor's login page.

```js
const config = {
  serverUrl: "https://door.casbin.com",
  clientId: "014ae4bd048734ca2dea",
  organizationName: "casbin",
  appName: "app-casnode",
  redirectPath: "/callback",
};

xxx.initCasdoorSdk(config);
```

**(Optional)** Because we are using React as example, so our `/callback` path is hitting the React route, we use the following React component to receive the `/callback` call and sends to the backend. You can ignore this step if you are redirecting to backend directly (like in JSP or PHP).

```js
import React from "react";
import { Button, Result, Spin } from "antd";
import { withRouter } from "react-router-dom";
import * as Setting from "./Setting";

class AuthCallback extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      classes: props,
      msg: null,
    };
  }

  componentWillMount() {
    this.login();
  }

  login() {
    Setting.signin().then((res) => {
      if (res.status === "ok") {
        Setting.showMessage("success", `Logged in successfully`);
        Setting.goToLink("/");
      } else {
        this.setState({
          msg: res.msg,
        });
      }
    });
  }

  render() {
    return (
      <div style={{ textAlign: "center" }}>
        {this.state.msg === null ? (
          <Spin
            size="large"
            tip="Signing in..."
            style={{ paddingTop: "10%" }}
          />
        ) : (
          <div style={{ display: "inline" }}>
            <Result
              status="error"
              title="Login Error"
              subTitle={this.state.msg}
              extra={[
                <Button type="primary" key="details">
                  Details
                </Button>,
                <Button key="help">Help</Button>,
              ]}
            />
          </div>
        )}
      </div>
    );
  }
}

export default withRouter(AuthCallback);
```

### 3. Get login URLs

Next you can show the "Sign up" and "Sign in" buttons or links to your users. The URLs can either be retrieved in the frontend or backend. See more details at: **[/docs/basic/core-concepts#login-urls](/docs/basic/core-concepts#login-urls)**

### 4. Get and verify access token

Here are the steps:

1. The user clicks the login URL and is redirected to Casdoor's login page, like: `https://door.casbin.com/login/oauth/authorize?client_id=014ae4bd048734ca2dea&response_type=code&redirect_uri=https%3A%2F%2Fforum.casbin.com%2Fcallback&scope=read&state=app-casnode`
2. The user enters username & password and clicks `Sign In` (or just click the third-party login button like `Sign in with GitHub`).
3. The user is redirected back to your application with the authorization code issued by Casdoor (like: `https://forum.casbin.com?code=xxx&state=yyy`), your application's backend needs to exchange the authorization code with the access token and verify that the access token is valid and issued by Casdoor. The functions `GetOAuthToken()` and `ParseJwtToken()` are provided by Casdoor backend SDK.

The following code shows how to get and verify the access token. For a real example of Casnode (a forum website written in Go), see: https://github.com/casbin/casnode/blob/6d4c55f5c9a3c4bd8c85f2493abad3553b9c7ac0/controllers/account.go#L51-L64

```go
// get code and state from the GET parameters of the redirected URL
code := c.Input().Get("code")
state := c.Input().Get("state")

// exchange the access token with code and state
token, err := auth.GetOAuthToken(code, state)
if err != nil {
	panic(err)
}

// verify the access token
claims, err := auth.ParseJwtToken(token.AccessToken)
if err != nil {
	panic(err)
}
```

If `ParseJwtToken()` finishes with no error, then the user has successfully logged into the application. The returned `claims` can be used to identity the user later.

### 4. Identify user with access token

:::info
This part is actually your application's own business logic and not part of OIDC, OAuth or Casdoor. We just provide good practices as a lot of people don't know what to do for next step.
:::


In Casdoor, access token is usually identical as ID token. They are the same thing. So the access token contains all information for the logged-in user.

The variable `claims` returned by `ParseJwtToken()` is defined as:

```go
type Claims struct {
	User
	AccessToken string `json:"accessToken"`
	jwt.RegisteredClaims
}
```

1. `User`: the User object, containing all information for the logged-in user, see definition at: **[/docs/basic/core-concepts#user](/docs/basic/core-concepts#user)**
2. `AccessToken`: the access token string.
3. `jwt.RegisteredClaims`: some other values required by JWT.

At this moment, the application usually has two ways to remember the user session: `session` and `JWT`.

#### Session

The Method to set session varies greatly depending on the language and web framework. E.g., Casnode uses [Beego web framework](https://github.com/beego/beego/) and set session by calling: `c.SetSessionUser()`.

```go
token, err := auth.GetOAuthToken(code, state)
if err != nil {
	panic(err)
}

claims, err := auth.ParseJwtToken(token.AccessToken)
if err != nil {
	panic(err)
}

claims.AccessToken = token.AccessToken
c.SetSessionUser(claims) // set session
```

#### JWT

The `accessToken` returned by Casdoor is actually a JWT. So if your application uses JWT to keep user session, just use the access token directly for it:

1. Send the access token to frontend, save it in places like localStorage of the browser.
2. Let the browser sends the access token to backend for every request.
3. Call `ParseJwtToken()` or your own function to verify the access token and get logged-in user information in your backend.

### 5. **(Optional)** Interact with the User table

:::info
This part is provided by `Casdoor Public API` and not part of the OIDC or OAuth.
:::

Casdoor Backend SDK provides a lot of helper functions, not limited to:

- `GetUser(name string)`: get a user by username.
- `GetUsers()`: get all users.
- `AddUser()`: add a user.
- `UpdateUser()`: update a user.
- `DeleteUser()`: delete a user.
- `CheckUserPassword(auth.User)`: check user's password.

These functions are implemented by making RESTful calls against `Casdoor Public API`. If a function is not provided in Casdoor Backend SDK, you can make RESTful calls by yourself.