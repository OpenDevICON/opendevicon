# Magic Bulb Frontend

This section provides an explanation about integrating the **Magic Link** authentication in your ICON React dApp using Magic SDK.

Magic interacts with the ICON blockchain via Magic's extension NPM package [**@magic-ext/icon**](https://www.npmjs.com/package/@magic-ext/icon). 
The ICON extension  lets you interact with the blockchain using methods from [**ICON's Javascript SDK**](https://www.icondev.io/docs/javascript-sdk).

## Initializing Extension

```javascript
import React, { useState, useEffect } from "react";
import { Magic } from "magic-sdk";
import { IconExtension } from "@magic-ext/icon";
import IconService from "icon-sdk-js";
 
const { IconBuilder, IconAmount, IconConverter } = IconService;
 
const magic = new Magic("pk_test_BAD78299B2E4EA9D", {
 extensions: {
   icon: new IconExtension({
     rpcUrl: "https://bicon.net.solidwallet.io/api/v3"
   })
 }
});

```
{% hint style="info" %}
For the private key **pk_test_BAD78299B2E4EA9D**, you need to create an account on [**magic-dashboard**](https://dashboard.magic.link/login). The user  account of the test key has the access of users that logged into the app.
{% endhint %}

## Logging In and Out

Using **getAccount()** function we get ICON public address for current user and  using **getMetadata()** function we get the user info.

**Login()** component checks if the user is logged in or not. If not, it prompts for a login with an email address. After the user clicks on login button providing the email address, magic-sdk sends a magic-link  to the user's email address. After clicking upon the link, the user is logged into the dApp. Then, user can click on logout button to logout from the dApp.

```javascript
export default function Login() {
   const [email, setEmail] = useState("");
   const [publicAddress, setPublicAddress] = useState("");
   const [isLoggedIn, setIsLoggedIn] = useState(false);
   const [userMetadata, setUserMetadata] = useState({});
 
   useEffect(() => {
     magic.user.isLoggedIn().then(async magicIsLoggedIn => {
       setIsLoggedIn(magicIsLoggedIn);
       if (magicIsLoggedIn) {
         const publicAddress = await magic.icon.getAccount();
         setPublicAddress(publicAddress);
         setUserMetadata(await magic.user.getMetadata());
       }
     });
   }, [isLoggedIn]);
 
    const login = async () => {
     await magic.auth.loginWithMagicLink({ email });
     setIsLoggedIn(true);
   };
 
    const logout = async () => {
     await magic.user.logout();
     setIsLoggedIn(false);
   };

   return (
     <div className="login">
       {!isLoggedIn ? (
         <div className="container">
           <h1>Please sign up or login</h1>
           <input
             type="email"
             name="email"
             required="required"
             placeholder="Enter your email"
             onChange={event => {
               setEmail(event.target.value);
             }}
           />
           <button onClick={login}>Send</button>
         </div>
       ) : (
         <div>
           <div className="container">
             <h1>Current user: {userMetadata.email}</h1>
             <button onClick={logout}>Logout</button>
           </div>
           <div className="container">
             <h1>Icon address</h1>
             <div className="info">
               <a
                 href={`https://bicon.tracker.solidwallet.io/address/${publicAddress}`}
                 target="_blank"
               >
                 {publicAddress}
               </a>
             </div>
           </div>
      
         </div>
       )}
     </div>
   );
}


```

## Signing a transaction

You can use magic to sign the transaction for the user.

```javascript
// Building a transaction
const txObj = new IconBuilder.IcxTransactionBuilder()
  .from(metadata.publicAddress)
  .to(destinationAddress)
  .value(IconAmount.of(sendICXAmount, IconAmount.Unit.ICX).toLoop())
  .stepLimit(IconConverter.toBigNumber(100000))
  .nid(IconConverter.toBigNumber(3))
  .nonce(IconConverter.toBigNumber(1))
  .version(IconConverter.toBigNumber(3))
  .timestamp(new Date().getTime() * 1000)
  .build();

// Sending a transaction
const txhash = await magic.icon.sendTransaction(txObj);

console.log(`Transaction Hash: ${txhash}`);

```