# MSMC

A bare bones login library for Minecraft based projects to authenticate individuals with a Microsoft account.

# credit

Based off the Oauth2 flow outline on <a href="https://wiki.vg/Microsoft_Authentication_Scheme"> this site</a>

# Examples

### Electron code sample:

This is a code sample for electron. It should be added to your main.js file. This will launch a popup that allows a user to log in with their microsoft account as soon as possible. Fastlaunch actually emulates the vanilla minecraft launcher, meaning that we can use mojangs own client ID to login. Inline login windows should use the older method.

```js
app.whenReady().then(() => {
  ...
  require("msmc").getElectron().FastLaunch(
    (call) => {
      // The function is called when the login has been successful
      console.log("Login successful");
      var accessToken = call.access_token;
      var profile = call.profile;
    },
    (update) => {
      // A hook for catching loading bar events and errors
      // Possible types are: Starting, Loading, Rejection, Error
      switch (update.type) {
        case "Starting":
          console.log("Checking user started!");
          break;
        case "Loading":
          console.log("Loading:", update.data, "-", update.percent + "%");
          break;
        case "Rejection":
          console.error("Fetch rejected!", update.data);
          break;
        case "Error":
          console.error("MC-Account error:", update.data);
          break;
      }
    }
  )
 ...
})
```

### NW.js code sample:

It is recommended to run this in NW.js's node context. Basically in a library that is called via a "require" call in a browser thread.

```js
require("msmc").getNWjs().FastLaunch(
    (call) => {
      // The function is called when the login has been successful
      console.log("Login successful");
      var accessToken = call.access_token;
      var profile = call.profile;
    },
    (update) => {
      // A hook for catching loading bar events and errors
      // Possible types are: Starting, Loading, Rejection, Error
      switch (update.type) {
        case "Starting":
          console.log("Checking user started!");
          break;
        case "Loading":
          console.log("Loading:", update.data, "-", update.percent + "%");
          break;
        case "Rejection":
          console.error("Fetch rejected!", update.data);
          break;
        case "Error":
          console.error("MC-Account error:", update.data);
          break;
      }
    }
)
```

# Docs

## Functions

An override to manually define which version of fetch should be used. Useful for if you have multiple versions of fetch available and want to use a specific variant<br>
fetchIn => A version of fetch <br>

`setFetch(fetchIn: any): void`<br>

token => Basic MS token info<br>
callback => The callback that is fired on a successful login. It contains a mojang access token and a user profile<br>
updates => A callback that one can hook into to get updates on the login process<br>
returns => The URL needed to log in your user in the form of a promise. You need to send this to a web browser or something similar to that!<br>

(RECOMMENDED) <br>
`MSLogin(token: MSToken, callback: (info: callback) => void, updates?: (info: update) => void): Promise<string>`<br>

An exposed version of the function that gets called when this library has found a login code. You can use this for custom setups where you do not want to use premade functions provided by the library for yout stuff.
code => The code gotten from a successful login <br>
MStoken => The MS token object <br>
callback => The callback that is fired on a successful login. It contains a mojang access token and a user profile<br>
updates => The URL needed to log in your user. You need to send this to a web browser or something similar to that!<br>

`MSCallBack(code: string, MStoken: MSToken, callback: (info: callback) => void, updates?: (info: update) => void): Promise<void>;`<br>

Use with electron to get a electron version of fast launch <br>
returns => <br>
{ <br>
&nbsp;&nbsp;&nbsp;&nbsp;callback => The callback that is fired on a successful login. It contains a mojang access token and a user profile<br>
&nbsp;&nbsp;&nbsp;&nbsp;updates => A callback that one can hook into to get updates on the login process<br>
&nbsp;&nbsp;&nbsp;&nbsp;properties => See windowProperties interface for more information<br>
&nbsp;&nbsp;&nbsp;&nbsp;`FastLaunch: (callback: (info: callback) => void, updates?: (info: update) => void,properties?:windowProperties)=>void` <br>
}<br>

`getElectron()` or `getNWjs()` <br>

This function will create a login link based on the inputs provided. Note that this function is called internally after the redirect has been formated. Aka after "http://localhost:\<port\>/" is appended to the redirect. This is done to allow us to create the "FastLaunch" methods which don't rely on an internal http server<br>

token => The MS token object <br>

`CreateLink(token: MSToken):String` <br>

## interfaces

### MSToken:

The Oauth2 details needed to log you in.

Resources

1. https://docs.microsoft.com/en-us/graph/auth-register-app-v2
2. https://docs.microsoft.com/en-us/graph/auth-v2-user#1-register-your-app
3. https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps
4. https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code

Recommendations:

1. Use "Mobile and desktop applications" as your type setting and make sure to set it up to only use "Personal Microsoft accounts only".
   You're not a university!

2. set the redirect to "http://localhost/...", With localhost specifically Microsoft does not check port numbers.
   This means that http://localhost:1/... to http://localhost:65535/... are all the same redirect to MS. (http://localhost/... == http://localhost:80/... btw)
   This app does not allow you to set the port manually, due to the extreme risk of unforseen bugs popping up.

3. If you set the ridirect to, for example, "http://localhost/Rainbow/Puppy/Unicorns/hl3/confirmed" then the variable {redirect} needs to equal "Rainbow/Puppy/Unicorns/hl3/confirmed".

4. Basically the redirect field is equal to your redirect URL you gave microsoft without the "http://localhost/" part.
   Please keep this in mind or you'll get weird errors as a mismatch here will still work...sort of.

```ts
interface MSToken {
    client_id: string,
    clientSecret?: string,
    redirect?: string,
    prompt?: "login" | "none" | "consent" | "select_account"
}
```

### callback:

Used in the callback that is fired upon a successfull login

access_token": string => Your classic Mojang auth token. You can do anything with this that you could do with the normal MC login token <br>
profile: { "id": string, "name": string, "skins": [], "capes": [] } => Player profile. Similar to the one you'd normaly get with the mojang login

```ts
interface callback {
    "access_token": string,
    profile: { "id": string, "name": string, "skins": [], "capes": [] }
}
```

### update:

Used in the callback that is fired multiple times during the login process to give the user feedback on how far along the login process is

```ts
interface update {
    type: "Loading" | "Rejection" | "Error" | "Starting" | "Canceled", //See table below!
    /**Some information about the call. Like the component that's loading or the cause of the error. */
    data?: string,
    /**Used by the rejection type.*/
    response?: Response,
    /**Used to show how far along the object is in terms of loading*/
    percent?: number
}
```

Possible values for the 'type' parameter:

 <table>
    <tr>
 <th>Value</th>
 <th>Cause</th>
  </tr>
 <td>"Loading" </td>
 <td>This gives input with regards to how far along the login process is </td>
  </tr>
   <tr>
<td> "Rejection" </td>
 <td>This is given with a fetch error. You are given the fetch item as a data object.  </td>
  </tr>
   <tr>
 <td>"Error"</td>
  <td>This is given with a normal MC account error and will give you some user readable feedback. </td>
      </tr>
        <tr>
 <td>"Starting"</td>
  <td>This is fired once when the whole loading process is started. This is mainly for setting up loading bars and stuff like that. </td>
      <tr>
  <tr>
   <tr>
 <td>"Canceled"</td>
  <td>When the user closes out of a popup (Electron / NV.js / methods that involve a GUI only) . </td>
      </tr>
   </table>

### WindowsProperties
Window property is set to any to avoid needing both nw.js and electron loaded as dev dependencies<br>
The common properties between both libraries has been backed into the type informarmation for this interface however<br> 
See <a href="https://www.electronjs.org/docs/api/browser-window#new-browserwindowoptions">this</a> resource for Electron, if you want to know what options are available <br>
See <a href="https://nwjs.readthedocs.io/en/latest/References/Manifest%20Format/#window-subfields">this</a> resource for NW.js, if you want to know the available options<br>

```ts
interface WindowsProperties {
    width: number,
    height: number,
    resizable?: boolean,
    [key: string]: any
}
```

### FastLaunchProperties:

Used with the gui based fastlaunch options. 
See the MStoken interface for more information on the 'prompt' property

```ts
FastLaunchProperties{
    prompt: "login" | "none" | "consent" | "select_account",
    window: WindowsProperties
}
```

###### Please report any type file bugs asap
