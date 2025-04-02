# TEConnect Payment Request (Angular)
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-ng.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-ng "@magensa/te-connect-ng npm.js")  
Angular component for use with Apple Pay and Google Pay via Token Exchange Connect.
TypeScript Type Defs are exposed, and can be imported from this library as well.

## `te-connect-ng` v2 (Angular ^19)
This `main` branch details implementation instructions for `te-connect-ng` v2, which has migrated to `"standalone"` components, and requires Angular ^19. All `te-connect-ng` v1 versions use NgModules. Documentation for v1 can be found in this repo, on the legacy `master` branch.

# Payment Request Platforms
TEConnect currently offers two payment request platforms to create payment tokens: [Apple Pay](./TecApplePayREADME.md), and [Google Pay](./TecGooglePayREADME.md).  The payment tokens are processed via [Magensa's Payment Protection Gateway](https://svc1.magensa.net/MPPGv4/MPPGv4Service.svc) (MPPG). There are many configurable options exposed when using [Apple Pay](./TecApplePayREADME.md) and/or [Google Pay](./TecGooglePayREADME.md) via TEConnect.  This document will focus on a simple implementation using both platforms. If you are interested in a more customized approach, using [Apple Pay](./TecApplePayREADME.md) and/or [Google Pay](./TecGooglePayREADME.md) - check out the [TecApplePayREADME.md](./TecApplePayREADME.md), and/or [TecGooglePayREADME](./TecGooglePayREADME.md) which will cover more specific configuration for each - as well as links to Apple/Google's documentation, respectively.

# Getting Started
Include Token Exchange Connect (core logic) and Token Exchange Connect Angular library in your workspace.
```
npm install @magensa/te-connect @magensa/te-connect-ng
```

If you would prefer to let the code speak, below we have an [example implementation](#Example-Implementation) for both platforms - along with a [Google Pay complex example](./TecGooglePayREADME.md#Google-Pay-Example-Implementation) and an [Apple Pay complex example](./TecApplePayREADME.md#Apple-Pay-Example-Implementation).

# Step-by-step
1. The first step is to create a TEConnect instance, and feed that instance to the `TecPaymentRequestComponent` via the `[teConnect]` property:
    - The first parameter for the `createTEConnect` function is your public key (`string`) for [TEConnect manual entry](https://github.com/Magensa/te-connect-ng) (`TeConnectNgComponent`|`TecThreedsComponent` components).
    - The second parameter is the [```options``` object](./README.md#teconnect-options)
        - Supplying a `tecPaymentRequest` object to `options` will enable TecPaymentRequest.
        - For the ```tecPaymentRequest``` properties: 
            - supply your ```appleMerchantId``` to enable Apple Pay.
                - ```appleMerchantId``` is obtained after the on-boarding process with Magensa™ is completed. [Please reach out to Magensa™ support team for more info](https://www.magensa.net/support.html)
            - supply your ```googleMerchantId``` and `gatewayId` to enable Google Pay.
                - ```googleMerchantId``` is obtained after the [on-boarding process with Google](https://pay.google.com/business/console) is completed.
                - For Google Pay capabilities - ```googleMerchantId```, ```googleMerchantName```, and ```gatewayId``` are required.
                    - ```gatewayId``` is obtained after the on-boarding process with Magensa™ is completed. This is supplied to the [GooglePay Payment Request Object](./TecGooglePayREADME.md#Google-Pay-Payment-Request-Object).  If you need assistance obtaining a `gatewayId` please reach out to [Magensa™ support team for more info](https://www.magensa.net/support.html)
                    - `googleMerchantId` and `googleMerchantName` are obtained by registering with the [Google Pay and Wallet Console](https://pay.google.com/business/console). More info on [```MerchantInfo``` here](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo). 
                        - Be aware `googleMerchantId` is fed to the [```options``` object](./README.md#teconnect-options) - while the `gatewayId` is supplied to the [GooglePay Payment Request Object](./TecGooglePayREADME.md#Google-Pay-Payment-Request-Object)


```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect'
import { ApplePayPaymentRequest, GooglePaymentRequest, TecPaymentRequestObject } from '@magensa/te-connect-ng';


const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
        googleMerchantId: "__googleMerchantId__",
    }
});

const exampleApplePaymentRequestObject: ApplePayPaymentRequest = {
    storeDisplayName: "<-- Business Name -->",
    currencyCode: "USD",
    countryCode: "US",
    supportedNetworks: ['visa', 'masterCard', 'amex', 'discover', 'jcb'],
    merchantCapabilities: ['supports3DS'],
    total: {
        label: "Test Transaction",
        amount: "1.00",
        type: "final"
    }
};

const exampleGooglePaymentRequestObject: GooglePaymentRequest = {
	allowedCardNetworks: ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "MIR", "VISA"],
	merchantName: "<-- Google Pay and Wallet Console Business Name -->",
	gatewayId: "<-- Google GatewayId Provided by Magensa™ -->",
	transactionInfo: {
        totalPriceLabel: "Total",
        totalPriceStatus: 'FINAL',
        totalPrice: '1.00',
        currencyCode: 'USD',
        countryCode: 'US'
    }
}

const examplePaymentRequestObject: TecPaymentRequestObject = {
    applePay: exampleApplePaymentRequestObject,
    googlePay: exampleGooglePaymentRequestObject
}
```

2. In your component, there are four required parameters that must be declared to utilize the `TecPaymentRequestComponent`.  Those variables are:
- `[teConnect]`:
    - Your TEConnect instance, created via the `createTEConnect` method. You may share this instance with one of the [Manual Entry components](./README.md) as well.
- `(canMakePaymentsResult)`:
    - This event will emit a [`CanMakePaymentsResult`](#CanMakePaymentsResult) when the component mounts the DOM. Attach a listener function to capture this event.
- `[paymentRequestObject]`
    - Build a [```paymentRequestObject```](#Payment-Request-Object) to define the Payment Request Form(s).
        - [Apple's Payment Request Object Structure](./TecApplePayREADME.md#Payment-Request-Object).
        - [Google's Payment Request Object Structure](./TecGooglePayREADME.md#Payment-Request-Object).
    - Note that if you are using multiple platforms (i.e. both Apple and Google) you must seperate the payment request objects by platform (i.e. ```{ googlePay: {...}, applePay: {...}}```).
- `[confirmToken]`:
    - When a cardholder submits the payment request form - an event is emitted. Listen to the [```confirmToken```](#confirmToken-Event) event to inspect the result (as demonstrated below).
    - The ```completePayment``` function is a property of the `confirmToken` event _for Apple Pay only_. This function must be called within 30 seconds of the event firing - otherwise the Apple Pay form will timeout. For Google Pay - visible authorization (on the Google Pay form) is optional.

```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect'
import { 
    TecPaymentRequestComponent, 
    TEConnect, 
    TecPaymentRequestObject,
    CanMakePaymentsResult,
    ConfirmTokenListener,
    ConfirmTokenEvent
} from '@magensa/te-connect-ng';


const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
        googleMerchantId: "__googleMerchantId__",
    }
});

@Component({
  selector: 'app-root',
  imports: [ TecPaymentRequestComponent ],
  template: `
    <lib-tec-payment-request
      [teConnect] = "tecInstance"
      [paymentRequestObject] = "prObject"
      [confirmToken] = "confirmPaymentToken"
      (canMakePaymentsResult)="handleCanMakePaymentsResult($event)"
    ></lib-tec-payment-request>
  `,
  styleUrl: './app.component.css'
})
export class AppComponent { 
    tecInstance = TE_CONNECT;
    prObject: TecPaymentRequestObject = examplePaymentRequestObject;
    isAppleUser: boolean = false;

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods && availablePrMethods.applePay)
            this.isAppleUser = true;
    }

    confirmPaymentToken: ConfirmTokenListener = (tokenResponse: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResponse;

        if (error) {
            //For ApplePay - you can complete with string value, as below.
            if (typeof(completePayment) === 'function')
                completePayment("failure");
        }
        else {
            console.log(tokenDetails);
            if (typeof( completePayment ) === 'function') {
                //tokenDetails.type === 'applePay'
                completePayment('success');
                //pass `tokenDetails.token` to MPPG `ProcessTECApplePay` to process payment
            }
            else {
                //tokenDetails.type === 'googlePay'
                //pass `tokenDetails.token` to MPPG `ProcessGooglePay` to process payment
            }
        }
    }
}
```

The above is a minimally viable solution to get you started. There are three more code examples - a [viable example using both platforms](#example-implementation), Apple Pay and Google Pay, as well as two more complex examples to leverage more of the features available, for each respective platform ([Apple Pay Complex Example here](./TecApplePayREADME.md#Apple-Pay-Example-Implementation), [Google Pay Complex Example here](./TecGooglePayREADME.md#Google-Pay-Example-Implementation)). Read on for further features and details availble to the Payment Request Utility.  
<br />

# Payment Request Object
The Payment Request Object is required, in order to use the `TecPaymentRequestComponent` (`<lib-tec-payment-request />`). This object describes the form(s) that the end-user will interact with.  
Each platform will have a different payment request object with different properties. If your application has opted-in for more than one platform (i.e. both Google and Apple Pay) - be sure to seperate the payment request objects by platform:  

```typescript
const exampleApplePaymentRequestObject: ApplePayPaymentRequest = {
    storeDisplayName: "<-- Business Name -->",
    currencyCode: "USD",
    countryCode: "US",
    supportedNetworks: ['visa', 'masterCard', 'amex', 'discover', 'jcb'],
    merchantCapabilities: ['supports3DS'],
    total: {
        label: "Test Transaction",
        amount: "1.00",
        type: "final"
    }
};

const exampleGooglePaymentRequestObject: GooglePaymentRequest = {
	allowedCardNetworks: ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "MIR", "VISA"],
	merchantName: "<-- Google Pay and Wallet Console Business Name -->",
	gatewayId: "<-- Google GatewayId Provided by Magensa™ -->",
	transactionInfo: {
        totalPriceStatus: 'FINAL',
        totalPrice: '1.00',
        currencyCode: 'USD',
        countryCode: 'US'
    }
}

const examplePaymentRequestObject: TecPaymentRequestObject = {
    applePay: exampleApplePaymentRequestObject,
    googlePay: exampleGooglePaymentRequestObject
}
```

### Apple Pay Payment Request Object
[More information and helpful links about Apple's Payment Request Object can be found here](./TecApplePayREADME.md#Apple-Pay-Payment-Request-Object)

### Google Pay Payment Request Object
[More information and helpful links about Google's Payment Request Object can be found here](./TecGooglePayREADME.md#Google-Pay-Payment-Request-Object)

# Token Exchange Connect Payment Request Features
The Token Exchange Connect Payment Request Component (`TecPaymentRequestComponent` selector: `<lib-tec-payment-request />`) is the interface needed to interact, and respond to user's interactions on the Payment Request Form.

## ```[paymentRequestObject]```
Define your [payment request object](#Payment-Request-Object) with this variable. This object is required.

## ```(canMakePaymentsResult)```
This event will emit a [```CanMakePaymentsResult```](#CanMakePaymentsResult). Define a listener function to listen to this event. This event fires under two circumstances:
- Once the component is mounted.
- If the [```paymentRequestObject```](#Payment-Request-Object) is updated.

#### ```CanMakePaymentsResult```
```typescript
type CanMakePaymentsResult = {
    applePay?: boolean,
    googlePay?: boolean
} | null;
```

## ```[confirmToken]```
Define your listener function for the [`confirmToken`](#confirmToken-Event) event with this variable. This function is required.

## Update Payment Request Object
Once you have supplied a [payment request object](#Payment-Request-Object) as the component's `[paymentRequestObject]` - this object will be watched, and can be updated as needed. Be aware of a few things about updating your `[paymentRequestObject]`:
- Updates to the payment request object are only useful __before__ the ApplePay/GooglePay Button(s) is displayed.
    - Once the user has the ability to hit the button - the payment request object is defined for that session.
    - When the object is updated - this is a __full update__, so the object will completely replace the previous payment request object supplied.  
- It's not necessary (and not recommended) to update your payment request object(s) inside of [payment request listeners](#Payment-Request-Event-Handlers). Any response in your completion functions will update the form dynamically.  


## ```[tecPaymentRequestOptions]```
Use this variable to define your custom [Payment Request Button Options](#Payment-Request-Button-Options). This value is optional.

## ```[shippingContactUpdate]``` __(Apple Pay Only)__
Use this variable to define your [Shipping Contact](./TecApplePayREADME.md#shippingContactUpdate-Event) event listener function. This value is optional.

## ```[paymentMethodSelection]``` __(Apple Pay Only)__
Use this variable to define your [Payment Method](./TecApplePayREADME.md#paymentMethodSelection-Event) event listener function. This value is optional.

## ```[shippingMethodUpdate]``` __(Apple Pay Only)__
Use this variable to define your [Shipping Method](./TecApplePayREADME.md#shippingMethodUpdate-Event) event listener function. This value is optional.

## ```[cancelTransaction]``` __(Apple Pay Only)__
Use this variable to define your [Cancel Transaction](./TecApplePayREADME.md#cancelTransaction-Event) event listener function. This value is optional.


# Payment Request Event Handlers
Listening to the `confirm-token` event is required to complete the workflow for all payment platforms.

Apple Pay uses an event handler driven workflow. When users interact with the payment request form - there are several events that are fired.   
[More details about Apple Pay listeners here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)
  
### ```[confirmToken]``` Event
This is the only event listener that is required to complete both workflows. This is the only event that can optionally contain an ```error``` property (in the case the payment was submitted, but was unsuccessful). Be sure to check for that property first, if it exists.  
When listening to the ```confirmToken``` event - there will be up to two special properties to the event:
- ```tokenDetails```
    - ```type``` specifies the payment request platform in which the token was created.
        - ```applePay``` or ```googlePay```.
    - ```token``` will be the object needed to process transactions, using the payment token created during the current session.
        - Pass the ```token``` to the appropriate MPPG operation, unaltered, for processing.
            - `ProcessTECApplePay` for ApplePay.
            - `ProcessGooglePay` for GooglePay.
    - ```error``` is only present in the case the token creation was unsuccessful.
- ```completePayment```
    - This function will only exist for Apple Pay tokens. It is used to close the Apple Pay form with a status message. 
        - [More information about ```completePayment``` can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)
        - This property will not exist for Google Pay tokens. This is one of the many ways to differentiate between which platform the user is using to complete the transaction. 
            
```confirmToken``` example: 

```typescript
function confirmPaymentToken(tokenResp: ConfirmTokenEvent) : void {
    const { tokenDetails, completePayment, error } = tokenResponse;

    if (error) {
        //You can complete with string value, as below.
        //completePayment("failure");

        //Optionally - you can provide more concise error messages - to highlight the payment form
        if (typeof( completePayment ) === 'function') {
            completePayment({
                status: window['ApplePaySession'].STATUS_FAILURE,
                errors: [
                    new window['ApplePayError']("shippingContactInvalid", "postalCode", "ZIP Code is invalid"),
                    new window['ApplePayError']("addressUnserviceable", "addressLines", "Cannot deliver to P.O. Box")
                ]
            });
        }
    }
    else {
        if (typeof( completePayment ) === 'function') {
            //tokenDetails.type === 'applePay'
            completePayment('success');
            //pass `tokenDetails.token` to MPPG to process payment
        }
        else {
            //tokenDetails.type === 'googlePay'
            //pass `tokenDetails.token` to MPPG to process payment
        }
    }
}
```

# Payment Request Button Options
The `[tecPaymentRequestOptions]` is an optional property.  Here you can define your custom options (`TecPaymentRequestOptions`) for the payment request buttons. You may define custom values for the Apple Pay Button, using the property `applePayOptions`(`ApplePayButtonOptions`), and Google Pay Button with `googlePayOptions`(`GooglePayButtonOptions`).  

You can see the [default values](#Default-Payment-Request-Button-Options-Values), for an example on how to structure the object, below.

## Apple Pay Button Options
[More details about Apple Pay Button Options can be found here](./TecApplePayREADME.md#Apple-Pay-Button-Options)

## Google Pay Button Options
[More details about Google Pay Button Options can be found here](./TecGooglePayREADME.md#Google-Pay-Button-Options)

#### Default Payment Request Button Options Values:  
  
```typescript
const applePayButtonOptions: ApplePayButtonOptions = {
    buttonLanguage: 'en',
    buttonType: 'plain',
    buttonStyle: 'black'
};

const googlePayButtonOptions: GooglePayButtonOptions = {
    preClick: undefined, //Only calls if is of type 'function'
    buttonColor: "default", //Currently defaults to 'black' but default is not static, and is determined by Google
    buttonType: "buy", 
    buttonLocale: undefined,  //If not supplied - defaults to browser or OS language settings
    buttonSizeMode: 'static',
    buttonRadius: undefined //number - representing pixels for button radius, if supplied
};

const defaultButtonOptions: TecPaymentRequestOptions = {
    applePayOptions: applePayButtonOptions,
    googlePayOptions: googlePayButtonOptions
};
```

# Example Implementation
```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect'
import { 
    TecPaymentRequestComponent, 
    TEConnect, 
    TecPaymentRequestObject,
    CanMakePaymentsResult,
    ConfirmTokenListener,
    ConfirmTokenEvent,
    TecPaymentRequestOptions
} from '@magensa/te-connect-ng';
import { examplePaymentRequestObject } from '../config';


const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
        googleMerchantId: "__googleMerchantId__",
    }
});


@Component({
  selector: 'app-root',
  imports: [ TecPaymentRequestComponent ],
  template: `
    <lib-tec-payment-request
      [teConnect] = "tecInstance"
      [paymentRequestObject] = "prObject"
      [confirmToken] = "confirmPaymentToken"
      (canMakePaymentsResult)="handleCanMakePaymentsResult($event)"
      [tecPaymentRequestOptions]="buttonOptions"
    ></lib-tec-payment-request>
  `,
  styleUrl: './app.component.css'
})
export class AppComponent { 
    tecInstance = TE_CONNECT;
    isAppleUser: boolean = false; //This flag is optional, but helps with session context
    prObject: TecPaymentRequestObject = examplePaymentRequestObject; //See examples above
    //Button options are optional
    buttonOptions: TecPaymentRequestOptions = {
        applePayOptions: {
            buttonLanguage: 'en-US',
            buttonType: 'plain',
            buttonStyle: 'black'
        },
        googlePayOptions: {
            buttonType: "pay",
            buttonRadius: 2
        }
    };

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods && availablePrMethods.applePay)
            this.isAppleUser = true;
    }

    confirmPaymentToken: ConfirmTokenListener = (tokenResponse: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResponse;

        if (error) {
            //For ApplePay - you can complete with string value, as below.
            if (typeof(completePayment) === 'function') {
                //completePayment("failure");

                //Optionally - you can provide more concise error messages - to highlight the payment form
                if (this.isAppleUser === true) {
                    completePayment({
                        status: window['ApplePaySession'].STATUS_FAILURE,
                        errors: [
                            new window['ApplePayError']("shippingContactInvalid", "postalCode", "ZIP Code is invalid"),
                            new window['ApplePayError']("addressUnserviceable", "addressLines", "Cannot deliver to P.O. Box")
                        ]
                    })
                }
            }
        }
        else {
            console.log(tokenDetails);
            if (typeof( completePayment ) === 'function') {
                //tokenDetails.type === 'applePay'
                completePayment('success');
                //pass `tokenDetails.token` to MPPG `ProcessTECApplePay` to process payment
            }
            else {
                //tokenDetails.type === 'googlePay'
                //pass `tokenDetails.token` to MPPG `ProcessGooglePay` to process payment
            }
        }
    }
}
```

# Payment Request Error Handling
All Apple Pay callbacks accept an ```errors[]``` array. [More information here](./TecApplePayREADME.md#Apple-Pay-Error-Handling)  
  
Google Pay Errors are handled within the optional callbacks. [More information on that here](./TecGooglePayREADME.md#Google-Pay-Error-Handling)
