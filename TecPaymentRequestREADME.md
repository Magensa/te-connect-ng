# TEConnect Payment Request (Angular)
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-ng.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-ng "@magensa/te-connect-ng npm.js")  
Angular components for use with Apple Pay and Google Pay via Token Exchange Connect.
TypeScript Type Defs are exposed, and can be imported from this module as well.

# Payment Request Platforms
TEConnect currently offers two payment request platforms to create payment tokens: [Apple Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md), and [Google Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md).  The payment tokens are processed via [Magensa's Payment Protection Gateway](https://svc1.magensa.net/MPPGv4/MPPGv4Service.svc) (MPPG). To that end - there are many configurable options exposed when using [Apple Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md), and [Google Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md) via TEConnect.  This document will focus on a very simple implementation using both platforms. If you are interested in a more customized experience for your users, using [Apple Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md), and/or [Google Pay](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md) - check out the [TecApplePayREADME.md](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md), and/or [TecGooglePayREADME](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md) which will cover more specific configuration for each - as well as links to Apple/Google's documentation, respectively.

# Getting Started
```
npm install @magensa/te-connect @magensa/te-connect-ng
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-ng
```

If you would prefer to let the code speak, below we have an [example implementation](#Example-Implementation) for both platforms - along with a [Google Pay complex example](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Google-Pay-Example-Implementation) and an [Apple Pay complex example](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Example-Implementation)

# Step-by-step
1. The first step is to create a TEConnect instance, and feed that instance into the TEConnectModuleNg where you are importing it:
    - The first parameter is your public key for [TEConnect manual entry](https://github.com/Magensa/te-connect-ng) (```CardEntry``` component).
    - The second parameter is the [```options``` object](https://github.com/Magensa/te-connect-ng#teconnect-options)
        - For the ```tecPaymentRequest``` options: 
            - supply your ```appleMerchantId``` to enable Apple Pay.
                - ```appleMerchantId``` is obtained after the on-boarding process with Magensa™ is completed. [Please reach out to Magensa™ support team for more info](https://www.magensa.net/support.html)
            - supply your ```googleMerchantId``` to enable Google Pay.
                - ```googleMerchantId``` is obtained after the [on-boarding process with Google](https://pay.google.com/business/console) is completed.
                - For Google Pay capabilities - ```googleMerchantId```, ```googleMerchantName```, and ```gatewayId``` are needed.
                    - ```gatewayId``` is obtained after the on-boarding process with Magensa™ is completed. [Please reach out to Magensa™ support team for more info](https://www.magensa.net/support.html)
                    - ```googleMerchantId``` and ```googleMerchantName``` are obtained using [Google Pay and Wallet Console](https://pay.google.com/business/console). [More info on ```MerchantInfo``` here](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo). Be aware that the name and id are fed to TEConnect seperately - which differs from the direct integration [Google Pay documentation](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo).


```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule, TEConnect } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect'

import { AppComponent } from './app.component';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
        googleMerchantId: "__googleMerchantId__",
    }
});

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    TeConnectNgModule.forRoot(TE_CONNECT)
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

2. In your component, there are three variables that must be declared to utilize the Payment Request workflow.  Those variables are:
- ```(canMakePaymentsResult)```:
    - This event will emit a [```CanMakePaymentsResult```](#CanMakePaymentsResult) when the component mounts the DOM. Attach a listener function to capture this event.
- ```[paymentRequestObject]```
    - Build a [```paymentRequestObject```](#Payment-Request-Object) to define the Payment Request Form(s).
    - [Apple's Payment Request Object Structure](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Payment-Request-Object), [Google's Payment Request Object Structure](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Payment-Request-Object).
    - Note that if you are using multiple platforms (i.e. both Apple and Google) you must seperate the payment request objects by platform (i.e. ```{ googlePay: {...}, applePay: {...}}```).
- ```[confirmToken]```:
    - When a user submits the payment request form - an event is emitted. Listen to the [```confirmToken```](#confirmToken-Event) event to inspect the result (as demonstrated below).
    - The ```completePayment``` function is a property of the ```confirmToken``` event _for Apple Pay only_. This function must be called within 30 seconds of the event firing - otherwise the Apple Pay form will timeout.  For Google Pay - visible authorization (on the Google Pay form) is optional.

```typescript
import { Component } from '@angular/core';
import { 
    TecPaymentRequestObject,
    CanMakePaymentsResult,
    ConfirmTokenEvent
} from 'te-connect-ng';

import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

@Component({
  selector: 'app-root',
  templateUrl: `
    <lib-tec-payment-request
        [confirmToken] = "confirmPaymentToken"
        [paymentRequestObject] = "prObject"
        (canMakePaymentsResult) ="handleCanMakePaymentsResult($event)"
    ></lib-tec-payment-request>
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {

    prObject: TecPaymentRequestObject = {
        applePay: exampleApplePaymentRequestObject,
        googlePay: exampleGooglePaymentRequestObject
    };

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods) {
            //const { applePay, googlePay } = availablePrMethods;
        }
    }

    confirmPaymentToken = (tokenResp: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResp;

         if (error) {
           //Unsuccessful - check message.
            if (typeof(completePayment) === 'function')
                completePayment('failure');
        }
        else {
            //Successful
            //Pass `tokenDetails.token` directly to MPPG for processing.
            //if Apple Pay - close user's payment request form with payment success notification.
            if (typeof(completePayment) === 'function')
                completePayment('success');
        }
    }
}
```

The above is a minimally viable solution to get you started. There are three more code examples - a [minimally viable example](#example-implementation) using both Apple Pay and Google Pay, as well as two more complex examples to leverage more of the features available, for each respective platform ([Apple Pay Complex Example here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Example-Implementation), [Google Pay Complex Example here](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Google-Pay-Example-Implementation)). Read on for further features and details availble to the Payment Request Utility.  
<br />

# Payment Request Object
The Payment Request Object is required, in order to use the ```lib-tec-payment-request```. This object describes the form(s) that the end-user will interact with.  
Each platform will have a different payment request object with different properties. If your application has opted-in for more than one platform (i.e. both Google and Apple Pay) - be sure to seperate the payment request objects by platform:  

```javascript
const applePaymentRequestObject = {
    storeDisplayName: "TEConnect Example Store",
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

const googlePaymentRequestObject = {
    allowedCardNetworks: ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "MIR", "VISA"],
	merchantName: "TEConnect Example Merchant",
	gatewayId: "_gateway_id_provided_by_Magensa_",
	transactionInfo: {
		totalPriceStatus: 'FINAL',
		totalPrice: '1.23',
		currencyCode: 'USD',
		countryCode: 'US'
    }
};

const teConnectPaymentRequestObject = {
    applePay: applePaymentRequestObject,
    googlePay: googlePaymentRequestObject
};
```

### Apple Pay Payment Request Object
[More information and helpful links about Apple's Payment Request Object can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Payment-Request-Object)

### Google Pay Payment Request Object
[More information and helpful links about Google's Payment Request Object can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Google-Pay-Payment-Request-Object)

# Token Exchange Connect Payment Request Features
The Token Exchange Connect Payment Request Component is the interface needed to interact, and respond to user's interactions on the Payment Request Form.

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
Define your listener function for the [```confirmToken```](#confirmToken-Event) event with this variable. This function is required.

## Update Payment Request Object
Once you have supplied a [payment request object](#Payment-Request-Object) as the component's ```[paymentRequestObject]``` - this object will be watched, and can be updated as needed. Be aware of a few things about updating your ```[paymentRequestObject]```:
- Updates to the payment request object are only useful __before__ the Apple Pay Button is displayed.
    - Once the user has the ability to hit the button - the payment request object is defined for that session.
    - When the object is updated - this is a __full update__, so the object will completely replace the previous payment request object supplied.  
- It's not necessary to update your object inside of [payment request listeners](#Payment-Request-Event-Handlers). Any response in your completion functions will update the form dynamically.  


## ```[tecPaymentRequestOptions]```
Use this variable to define your custom [Payment Request Button Options](#Payment-Request-Button-Options). This value is optional.

## ```[shippingContactUpdate]``` __(Apple Pay Only)__
Use this variable to define your [Shipping Contact](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#shippingContactUpdate-Event) event listener function. This value is optional.

## ```[paymentMethodSelection]``` __(Apple Pay Only)__
Use this variable to define your [Payment Method](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#paymentMethodSelection-Event) event listener function. This value is optional.

## ```[shippingMethodUpdate]``` __(Apple Pay Only)__
Use this variable to define your [Shipping Method](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#shippingMethodUpdate-Event) event listener function. This value is optional.

## ```[cancelTransaction]``` __(Apple Pay Only)__
Use this variable to define your [Cancel Transaction](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#cancelTransaction-Event) event listener function. This value is optional.


# Payment Request Event Handlers
Listening to the ```confirm-token``` event is required to complete the workflow for all payment platforms.

Apple Pay uses an event handler driven workflow. When users interact with the payment request form - there are several events that are fired.   
[More details about Apple Pay listeners here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)
  
### ```confirmToken``` Event
This is the only event listener that is required to complete both workflows. This is the only event that can optionally contain an ```error``` property (in the case the payment was submitted, but was unsuccessful). Be sure to check for that property first, if it exists.  
When listening to the ```confirmToken``` event - there will be up to two special properties to the event:
- ```tokenDetails```
    - ```type``` specifies the payment request platform in which the token was created.
        - ```applePay``` or ```googlePay```.
    - ```token``` will be the object needed to process transactions, using the payment token created during the current session.
        - Pass the ```token``` to the appropriate MPPG operation, unaltered, for processing.
    - ```error``` is an optional property in the case the token creation was unsuccessful.
- ```completePayment```
    - This function will only exist for Apple Pay tokens. It is used to close the Apple Pay form with a status message. 
        - [More information about ```completePayment``` can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)
        - This property will not exist for Google Pay tokens. This is one of the many ways to differentiate between which platform the user is using to complete the transaction. 
            
```confirmToken``` example: 

```typescript
import { Component } from '@angular/core';
import { 
    TecPaymentRequestObject
    CanMakePaymentsResult,
    ConfirmTokenEvent
} from 'te-connect-ng';

import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

@Component({
  selector: 'app-root',
  templateUrl: `
    <lib-tec-payment-request
        [confirmToken] = "confirmPaymentToken"
        [paymentRequestObject] = "prObject"
        (canMakePaymentsResult)="handleCanMakePaymentsResult($event)"
    ></lib-tec-payment-request>
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
    isAppleUser: boolean = false;

    prObject: TecPaymentRequestObject = {
        applePay: exampleApplePaymentRequestObject,
        googlePay: exampleGooglePaymentRequestObject
    };

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods && availablePrMethods.applePay === true) {
            this.isAppleUser = true;
        }
    }

    confirmPaymentToken = (tokenResp: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResponse;

        if (error) {
            //You can complete with string value, as below.
            //completePayment("failure");

            //Optionally - you can provide more concise error messages
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
}
```

# Payment Request Button Options
The ```TecPaymentRequestButtons``` component accepts an optional prop named ```tecPrOptions```.  Here you can define your custom options for the payment request buttons. You may define custom values for the Apple Pay Button, using the property ```applePayOptions```, and Google Pay Button with ```googlePayOptions``` (You can see the [default values](#Default-Payment-Request-Button-Options-Values), for an example on how to structure the object, below).

## Apple Pay Button Options
[More details about Apple Pay Button Options can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Button-Options)

## Google Pay Button Options
[More details about Google Pay Button Options can be found here](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Google-Pay-Button-Options)

#### Default Payment Request Button Options Values:  
  
```typescript
const defaultButtonOptions: TecPaymentRequestOptions = {
    applePayOptions: {
        buttonLanguage: 'en',
        buttonType: 'plain',
        buttonStyle: 'black'
    },
    googlePayOptions: {
        preClick: undefined, //Only calls if is of type 'function'
        buttonColor: undefined, //Currently defaults to 'black' but default is not static, and is determined by Google
        buttonType: undefined, 
        buttonLocale: undefined, //If not supplied - defaults to browser or OS language settings
        buttonSizeMode: 'static'
    }
};
```

# Example Implementation
Below you will find a minimally viable implementation.  

```app.module.ts```
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule, TEConnect } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect';

import { AppComponent } from './app.component';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: { 
        appleMerchantId: "__tecAppleMerchantId__",
        googleMerchantId: "__googleMerchantId__"
    }
});

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    TeConnectNgModule.forRoot(TE_CONNECT)
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```app.component.ts```
```typescript
import { Component } from '@angular/core';
import { 
    CanMakePaymentsResult,
    TecPaymentRequestObject, 
    ConfirmTokenListener,
    ConfirmTokenEvent,
    CompletePaymentResult
} from 'te-connect-ng';

import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

@Component({
  selector: 'app-root',
  templateUrl: `
    <lib-tec-payment-request
        [paymentRequestObject] = "prObject"
        [confirmToken] = "confirmPaymentToken"
        (canMakePaymentsResult)="handleCanMakePaymentsResult($event)"
    ></lib-tec-payment-request>
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
    isAppleUser: boolean = false;

    prObject: TecPaymentRequestObject = {
        applePay: exampleApplePaymentRequestObject,
        googlePay: exampleGooglePaymentRequestObject
    };

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods && availablePrMethods.applePay) {
            this.isAppleUser = true;
        }
    }

    confirmPaymentToken: ConfirmTokenListener = (tokenResponse: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResponse;

        if (error) {
            //You can complete with string value, as below.
            //completePayment("failure");

            //Optionally - you can provide more concise error messages
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
}
```


# Payment Request Error Handling
All Apple Pay callbacks accept an ```errors[]``` array. [More information here](https://github.com/Magensa/te-connect-ng/blob/master/TecApplePayREADME.md#Apple-Pay-Error-Handling)  
  
Google Pay Errors are handled within the optional callbacks. [More information on that here](https://github.com/Magensa/te-connect-ng/blob/master/TecGooglePayREADME.md#Google-Pay-Error-Handling)
