[npm:img]: https://img.shields.io/npm/v/react-social-login.svg?style=flat-square
[npm:url]: https://www.npmjs.org/package/react-social-login
[standard:img]: https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square
[standard:url]: http://standardjs.com
[license:img]: https://img.shields.io/badge/License-MIT-brightgreen.svg?style=flat-square
[license:url]: https://opensource.org/licenses/MIT

# React Social Login &middot; [![NPM version][npm:img]][npm:url] &middot; [![Standard - JavaScript Style Guide][standard:img]][standard:url] &middot; [![Dependencies][dependencies:img]][dependencies:url] &middot; [![License: MIT][license:img]][license:url]

React Social Login is an HOC which provides social login through multiple providers.

**Currently supports Amazon, Facebook, GitHub, Google, Instagram and ~~LinkedIn~~ as providers (more to come!)**

\*<small>LinkedIn has deprecated it's JS SDK. Hence, not supported anymore.</small>

##### We aren't using Google+ Api. Library will work fine even after Google+ deprecation.

## Motivation

- Having a component that doesn't dictates the HTML
- All-in-One login component for different social providers
- Takes care of warnings from provider's SDKs when multiple instances are placed
- Kind of re-birth of my previous .Net driven similar open source - SocialAuth.NET

## Scopes

While this library does a good job of fetching basic profile, at times you might need to get additional attributes from providers like DateOfBirth, HomeTowm, etc. which aren't returned by default. Scopes are purely provider dependent and their documentation is the best place to look for supported scopes and literal to be passed as argument. You can pass those scopes using the scope tag. For example, if you want birth date from Facebook (which isn't returned by default), you'd add following scope to your tag:

```js
scope = "user_birthday,user_hometown";
```

Below are some links to official scopes guide for a few providers:

- [FACEBOOK](https://developers.facebook.com/docs/facebook-login/permissions/)
- [GOOGLE](https://developers.google.com/identity/protocols/googlescopes)

## Demo

Edit `appId` props with your own ones in `demo/index.js` file and build demo:

```shell
$ npm start
```

You can then view the demo at [https://localhost:8080][demo].

For GitHub provider, see [GitHub specifics][githubspecifics] first.

## Install

```shell
$ npm install --save react-social-login
```

## Usage

Create the component of your choice and transform it into a SocialLogin component.

**SocialButton.js**

```js
import React from "react";
import SocialLogin from "react-social-login";

class SocialButton extends React.Component {
  render() {
    const { children, triggerLogin, ...props } = this.props;
    return (
      <button onClick={triggerLogin} {...props}>
        {children}
      </button>
    );
  }
}

export default SocialLogin(SocialButton);
```

Then, use it like a normal component.

**index.js**

```js
import React from "react";
import ReactDOM from "react-dom";

import SocialButton from "./SocialButton";

const handleSocialLogin = (user) => {
  console.log(user);
};

const handleSocialLoginFailure = (err) => {
  console.error(err);
};

ReactDOM.render(
  <div>
    <SocialButton
      provider="facebook"
      appId="YOUR_APP_ID"
      onLoginSuccess={handleSocialLogin}
      onLoginFailure={handleSocialLoginFailure}
    >
      Login with Facebook
    </SocialButton>
  </div>,
  document.getElementById("app")
);
```

### Optional Loader Component
There are times when initialization of a component can take longer than expected.
To support these changes there is the optional loader component.

Simply pass the component as a second parameter to SocialLogin

**SocialButton.js**

```js
import React from "react";
import SocialLogin from "react-social-login";
import Loading from "./Loading";

class SocialButton extends React.Component {
  render() {
    const { children, triggerLogin, ...props } = this.props;
    return (
      <button onClick={triggerLogin} {...props}>
        {children}
      </button>
    );
  }
}

export default SocialLogin(SocialButton, Loading);
```

### Reference

Raw component props (before transform):

| Prop              | Default | Type / Values                                                         | Description                                                                                                                |
| ----------------- | ------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| appId             | —       | string                                                                | Your app identifier (see [find my appId][findmyappid])                                                                     |
| autoCleanUri      | false   | boolean                                                               | Enable auto URI cleaning with OAuth callbacks                                                                              |
| autoLogin         | false   | boolean                                                               | Enable auto login on `componentDidMount`                                                                                   |
| gatekeeper        | —       | string                                                                | Gatekeeper URL to use for GitHub OAuth support (see [GitHub specifics][githubspecifics])                                   |
| getInstance       | —       | function                                                              | Return node ref like `ref` function would normally do ([react known issue](https://github.com/facebook/react/issues/4213)) |
| onLoginFailure    | —       | function                                                              | Callback on login fail                                                                                                     |
| onLoginSuccess    | —       | function                                                              | Callback on login success                                                                                                  |
| onLogoutFailure   | —       | function                                                              | Callback on logout fail (`google` only)                                                                                    |
| onLogoutSuccess   | —       | function                                                              | Callback on logout success                                                                                                 |
| onInternetFailure | —       | function                                                              | Doesn't open popup if returns false and internet isn't there                                                               |
| provider          | —       | `amazon`, `facebook`, `github`, `google`, `instagram`, ~~`linkedin`~~ | Social provider to use                                                                                                     |
| redirect          | -       | string                                                                | URL to redirect after login (available for `github` and `instagram` only)                                                  |
| scope             | -       | array, string                                                         | An array or string of scopes to be granted on login.                                                                       |
| version           | -       | string                                                                | Can be used to explicitly specify FBsdk version (default is v5.0)                                                          |
| any other prop    | —       | —                                                                     | Any other prop will be forwarded to your component                                                                         |

_Note about `redirect`: if you are redirecting on root (eg: https://localhost:8080), you **have** to omit the trailing slash._

Transformed component props:

| Prop           | Type     | Description                                                                              |
| -------------- | -------- | ---------------------------------------------------------------------------------------- |
| triggerLogin   | function | Function to trigger login process, usually attached to an event listener                 |
| triggerLogout  | function | Function to trigger logout process, usually attached to a container handling login state |
| all your props | —        | All props from your original component, minus SocialLogin specific props                 |

### Logout

To implement logout, we need a container handling login state and triggering logout function from a `ref` to `SocialLogin` component.

As it is implemented in the demo, we have two components working together to trigger logout:

- [`Demo` container][democontainer]
- [`UserCard` component][usercardcomponent]

Here is how they work together:

1.  [`Demo` is displaying `UserCard` only if user is logged][logoutstep1]
2.  [`UserCard` gets forwarded a `logout` function][logoutstep2]
3.  [`UserCard` calls forwarded `logout` prop on click on the logout button][logoutstep3]
4.  [`logout` function triggers `triggerLogout` prop from a ref to SocialLogin component][logoutstep4]

### Old component support

We decided to keep the old behavior as a fallback, it only supports `facebook`, `google` and `linkedin` providers and is available as a named export:

```js
import React from "react";
import ReactDOM from "react-dom";
import { OldSocialLogin as SocialLogin } from "react-social-login";

const handleSocialLogin = (user, err) => {
  console.log(user);
  console.log(err);
};

ReactDOM.render(
  <div>
    <SocialLogin
      provider="facebook"
      appId="YOUR_APP_ID"
      callback={handleSocialLogin}
    >
      <button>Login with Google</button>
    </SocialLogin>
  </div>,
  document.getElementById("app")
);
```

## Build

Though not mandatory, it is recommended to use latest npm5 to match lockfile versions.

```shell
$ npm install
$ npm run build
```

## Miscellaneous

### Find my appId

#### Amazon

See [Amazon developers documentation][amazondoc].

#### Facebook

See [facebook for developers documentation][fb4devdoc].

#### GitHub (see [GitHub specifics][githubspecifics])

- Basic authentication method using personal access tokens: see [GitHub Help][githubhelp].
- OAuth authentication: see [GitHub Developer guide][githubdoc].

#### Google

See [Google Sign-In for Websites guide][gsignindoc].

#### Instagram

See [Instagram developers documentation][instadoc].

#### LinkedIn

See `Where can I find my API key?` section on the [FAQ][linkedinfaq].

### GitHub specifics

GitHub provider is implemented in two different modes:

- One using [GitHub Personal Tokens][ghpersonaltokens]
- Another using GitHub OAuth

#### GitHub Personal Tokens mode

Actually, this one is more a hacky way to get user profile than a way to really _connect_ your app like OAuth does.

Plus, it requires from users to create their personal token from their GitHub account, which is not a good experience for them.

This mode is the default if you do not provide `gatekeeper` prop and will try to use the `appId` prop to get user data. Anyway, we **strongly advise you to use the GitHub OAuth authentication flow**.

#### GitHub OAuth

If you provide a `gatekeeper` prop, this mode will be active and will use a server of your own to fetch GitHub OAuth access token. This is a [know issue of GitHub][ghoauthwebflowissue].

The simplest way to setup this mode is to use the [Gatekeeper project][gatekeeper]. Just follow setup instructions then tell RSL to use it:

```
<SocialLogin
  provider='github'
  gatekeeper='http://localhost:9999'
  appId='YOUR_GITHUB_CLIENT_ID'
  redirect='http://localhost:8080'
>
  Login with GitHub OAuth
</SocialLogin>
```

You can also implement it your own way but you must use the same routing than `Gatekeeper` (`/authenticate/:code`) and return a JSON response containing a `token` or `error` property (it will also throw if it doesn't find `token`).

#### Special instructions to run demo

RSL demo is served over https with `webpack-dev-server`. This is a requirement of Amazon Login SDK. `Gatekeeper` is served over insecure http so you will have to serve the demo through http also to work with GitHub (but it will break Amazon):

```shell
$ npm run start:insecure
```

[demo]: https://localhost:8080
[findmyappid]: #find-my-appid
[fb4devdoc]: https://developers.facebook.com/docs/apps/register
[githubhelp]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line
[githubdoc]: https://developer.github.com/apps/building-integrations/setting-up-and-registering-oauth-apps/registering-oauth-apps
[gsignindoc]: https://developers.google.com/identity/sign-in/web/devconsole-project
[instadoc]: https://www.instagram.com/developer
[linkedinfaq]: https://developer.linkedin.com/support/faq
[ghpersonaltokens]: https://github.com/blog/1509-personal-api-tokens
[ghoauthwebflowissue]: https://github.com/isaacs/github/issues/330
[gatekeeper]: https://github.com/prose/gatekeeper
[githubspecifics]: #github-specifics
[amazondoc]: https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/register_web.html
[googlescopes]: https://developers.google.com/identity/protocols/googlescopes

