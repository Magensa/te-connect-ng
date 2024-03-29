# TEConnect Apple Pay (Angular)
This document demonstrates the available options, using Apple Pay via TEConnect.  TEConnect currently offers a Manual Entry form, Apple Pay and Google Pay.  Your app may use one - or all of these platforms to collect a payment token.  The section below explains how opt-in works with TEConnect, for each available platform. The remainder of the document focuses on Apple Pay with TEConnect.

# Payment Request Opt-In
```createTEConnect``` accepts a public key as the first argument. This public key is for [TEConnect Manual Entry](https://github.com/Magensa/te-connect-ng#Getting-Started).
The second argument is the [TEConnect options](https://github.com/Magensa/te-connect-ng#TEConnect-Options) object.  
Providing an ```appleMerchantId``` or a ```googleMerchantId``` to the ```tecPaymentRequest``` object is how to opt-in for each Payment Request Platform.  So providing an ```appleMerchantId``` is an opt-in for Apple Pay - and a ```googleMerchantId``` for Google Pay.  Example below.

```javascript
const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
        googleMerchantId: "__googleMerchantId__",
    }
});
```

Opting in to one or more platform affects the way the Payment Request Object should be supplied to TEConnect - as well as the  [```CanMakePaymentsResult```](TecPaymentRequestREADME.md#CanMakePaymentsResult).

Be aware that if you only opt-in for one platform (i.e. only an ```appleMerchantId``` is provided to ```createTEConnect```) - you may provide the payment request object for that specific platform.  In this case - the [Apple Pay](#Apple-Pay-Payment-Request-Object) can be provided as-is.  

In the case of multiple payment request platforms (i.e. both an ```appleMerchantId``` and ```googleMerchantId``` are provided) - the payment request object must be structured to reflect each platform.  Example below.

```javascript
const paymentRequestObject = {
    applePay: applePayRequestObject,
    googlePay: googlePayRequestObject
}
```


# Apple Pay Payment Request Object
The Payment Request Object is required, in order to use the ```lib-tec-payment-request```. This object describes the form that the end-user will interact with.
This section will specifically address the [ApplePayPaymentRequest object](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest).
- [Here is Apple's Documentation about the payment request object](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest)
- It is encouraged to read through Apple's documentation for all the options available to you to define your form.
- Be aware that any inaccuracy in the payment request object will most likely throw a ```TypeError``` without a message. Any errors that come from Apple's javascript rarely have any messages attached to errors.

Below is an example of a Payment Request object.  This is where your form is defined - so each customer's object will appear differently, depending on the customer's circumstances.
```javascript
const examplePaymentRequestObject = {
    storeDisplayName: "TEConnect Example Store",
    applePayVersion: 3, //Default is 3, unless specified
    currencyCode: "USD",
    countryCode: "US",
    supportedNetworks: ['visa', 'masterCard', 'amex', 'discover', 'jcb'],
    merchantCapabilities: ['supports3DS'],
    total: {
        label: "Test Transaction",
        amount: "1.00",
        type: "final"
    },
    shippingType: ['shipping'],
    shippingMethods: [
        {    
            label: 'Free Shipping',
            detail: "Arrives in a week",
            amount: '0.00',
            identifier: "FreeShipping"
        },
        {    
            label: 'Not Free Shipping',
            detail: "Arrives in less than week",
            amount: '10.00',
            identifier: "ChargeShipping"
        }
    ],
    requiredShippingContactFields: [
        'name',
        'email',
        'phone',
        'postalAddress'
    ],
    requiredBillingContactFields: [
        'name',
        'postalAddress'
    ],
    shippingContact: {
        phoneNumber: '888-8888',
        emailAddress: 'something@example.net',
        givenName: 'Bob',
        familyName: 'ExampleName',
        addressLines: ['123 Main St.', 'Suite 101'],
        locality: 'Los Angeles',
        administrativeArea: 'CA',
        postalCode: '90010',
        country: 'United States',
        countryCode: 'US',
    },
    billingContact: {
        emailAddress: 'something@example.net',
        givenName: 'Bob',
        familyName: 'ExampleName',
        addressLines: ['123 Main St.', 'Suite 101'],
        subLocality: 'Los Angeles',
        locality: 'CA',
        postalCode: '90010',
        country: 'United States',
        countryCode: 'US',
    }
};
```  

## Apple Pay Listeners
Listening to the ```confirm-token``` event is required to complete the workflow, but there are more events, for Apple Pay users, that may be optionally subscribed to.  

When [qualified Apple Pay users](#User-Requirements) interact with the Apple Pay form, there are several events that may be subscribed to. Each event will fire with an object (describing the event) and a function (to respond to the event). Be aware that _if_ an event is subscribed to - the response function _must_ be called with a response within 30 seconds - otherwise the form will timeout. The only exception to this is the ```cancelTransaction``` event - which only contains a message.  
Examples of all the listeners can be found in the more complex of the [Example Implementations](#Example-Implementation)

### ```confirmToken``` Event
This is the only event listener that is required to complete the Apple Pay workflow. This is the only event that can optionally contain an ```error``` property (in the case the payment was submitted, but was unsuccessful). Be sure to check for that property first, if it exists.  
When listening to the ```confirmToken``` event - there will be two special properties to the event:
- ```tokenDetails```
    - ```type``` specifies the payment request platform in which the token was created.
        - Currently ```applePay``` or ```googlePay```.
    - ```token``` will be the object needed to process transactions, using the payment token created during the current session.
        - Pass the ```token``` to the appropriate MPPG operation, unaltered, for processing.
    - ```error``` is an optional property in the case the token creation was unsuccessful.
- ```completePayment```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the Apple Pay form.
    - There are two possible value types to call this function with - either a string value, or an object:
        - This function expects either ```"sucess"``` or ```"failure"``` as string options - and will display the chosen completion status on the form.
        - Optionally, if you've confirmed the user is a [qualified Apple Pay user](#User-Requirements) (```canMakePayments``` result has returned ```{ applePay: true }```) you can provide an [Apple Pay Authorization Result](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentauthorizationresult) for more descriptive error messages.
            - In this case - there will be a ```status``` and an ```errors``` property. The status must be provided in a recognized format ([as demonstrated in the Apple docs](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentauthorizationresult)).
                - Since this is an error scenario it's likely that the value needed would be: ```ApplePaySession.STATUS_FAILURE```.
                - It's important to confirm that the user is a [qualified Apple Pay user](#User-Requirements) before providing this object - as ```ApplePaySession``` is a global variable that will not exist (throw an error) in all other browsing contexts.
            - The ```errors``` can be supplied as explained in the [Payment Request Error Handling](#Payment-Request-Error-Handling) section.  
            
```confirmToken``` example: 

```typescript
import { Component } from '@angular/core';
import { 
    ApplePayPaymentRequest,
    CanMakePaymentsResult,
    ConfirmTokenEvent
} from 'te-connect-ng';

import { examplePaymentRequestObject } from '../consts';

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

    prObject: ApplePayPaymentRequest = examplePaymentRequestObject;

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods) {
            //Positive CanMakePaymentsResult result
        }
    }

    confirmPaymentToken = (tokenResp: ConfirmTokenEvent) => {
        const { tokenDetails, completePayment, error } = tokenResponse;

        if (error) {
            //You can complete with string value, as below.
            //completePayment("failure");

            //Optionally - you can provide more concise error messages
            if (isAppleUser === true) {
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
            completePayment('success')
        }
    }
}
```

### ```paymentMethodSelection``` Event
When listening to the ```paymentMethodSelection``` event - there will be two special properties to the event:
- ```paymentMethod```
    - This object will provide details about the currently selected payment method. If you subscribe to this event - it will fire at least once (when the payment form loads), and will fire every time the user changes the payment method (during the current session). You must call ```completePaymentMethodSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethod).
- ```completePaymentMethodSelection```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the payment form.
    - A [Payment Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethodupdate) object is required to invoke this function. ```'newTotal'``` [line item](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaylineitem) will reflect the total cost of all the purchased items.
        - More information on the [Payment Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethodupdate) object.
        - More information on [line items](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaylineitem)
    - Only provide ```'newLineItems'``` if there are new or updated costs or discounts (depending on the payment method the user has selected) - Otherwise pass an empty array or ```null```.  


### ```shippingContactUpdate``` Event
When listening to the ```shippingContactUpdate``` event - there will be two special properties to the event:
- ```shippingContact```
    - This object will provide details about the currently selected shipping contact. If you subscribe to this event - it will fire every time the user changes their shipping contact (during the current session) - or if they edit their selected shipping contact mid-session. You must call ```completeShippingContactSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentcontact).
- ```completeShippingContactSelection```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the payment form.
    - A [Shipping Contact Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate) object is required to be supplied to this function.
        - More information about the [update object is here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate) 

### ```shippingMethodUpdate``` Event
When listening to the ```shippingMethodUpdate``` event - there will be two special properties to the event:
- ```shippingMethod```
    - This object will provide details about the currently selected shipping method. If you subscribe to this event - it will fire every time the user changes their shipping method (during the current session). You must call ```completeShippingMethodSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingmethod).
- ```completeShippingMethodSelection```
    - A [Shipping Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingmethodupdate) object is required to be supplied to this function.
        - More information about the [update object is here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate)

### ```cancelTransaction``` Event
When listening to the ```cancelTransaction``` event - there will only be a ```reason``` notifying you that the customer's payment request form has been dismissed.

# Apple Pay Button Options
There are many options and styles available to assist with tailoring the Apple Pay Button to your specifications. There are a few things to be aware of:
- Apple has defined [guidelines for displaying the Apple Pay button](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks)
    - The "Button Types" and "Button Styles" mentioned in the [guidelines](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks) can be supplied in the Apple Pay Button Options.
- ```buttonLanguage``` and ```buttonType``` values are not validated, and will be applied directly to the button's style properties. Please check the supplied value for accuracy. 
    - ```buttonStyle```, however, is validated, and will display an accurate value supplied - or default to ```'black'```, if the value is not recognized.
- In addition to the Apple Pay Button options available - it is also possible to apply your own custom styles to the button using CSS. Target the button's ```id```:
    - ```"te-connect-apple-pay-btn"```
    - When applying custom CSS - please be sure to adhere to [Apple's Guidelines](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks)

| Property Name | Type | Possible Values |
|:--:|:--:|:--:|
| ```buttonLanguage``` | ```string``` | [available button languages](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttonlocale) | 
```buttonType``` | ```string``` | [available button types](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttontype) |
| ```buttonStyles``` | ```string``` | ```'black'``` ```'white'``` ```'white-outline'``` [available button styles](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttonstyle)  |

## CSS Considerations
When the ```lib-tec-payment-request``` component mounts an Apple Pay button - it is wrapped in a ```<div>``` with the id of ```"te-connect-apple-pay-wrapper"```.  You can use this target to style the wrapper around the Apple Pay button, if needed.
  
<br />

# Apple Pay Example Implementation
The below implementation utilizes most of the Apple Pay features, for a more complex example. Your application's specific implementation will differ according to your needs.  

```app.module.ts```
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule, TEConnect } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect'

import { AppComponent } from './app.component';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        appleMerchantId: "__tecAppleMerchantId__"
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
import { CanMakePaymentsResult } from 'dist/te-connect-ng/lib/consts/paymentRequestTypeDefs';
import { 
    ApplePayPaymentRequest, 
    ConfirmTokenListener,
    PaymentMethodListener,
    ShippingMethodListener,
    ShippingContactListener,
    CancelListener,
    ConfirmTokenEvent,
    PaymentMethodEvent,
    ShippingMethodEvent,
    ShippingContactEvent,
    CancelEvent,
    CompletePaymentResult,
    ApplePayPaymentMethodUpdate,
    ApplePayShippingMethodUpdate,
    ApplePayShippingContactUpdate,
    TecPaymentRequestOptions
} from 'te-connect-ng';

import { examplePaymentRequestObject } from './consts';

@Component({
  selector: 'app-root',
  templateUrl: `
    <lib-tec-payment-request
        [paymentRequestObject] = "prObject"
        [confirmToken] = "confirmPaymentToken"
        [cancelTransaction] = "cancelTrx"
        [shippingMethodUpdate] = "shippingMethodHandler"
        [paymentMethodSelection] = "paymentMethodHandler"
        [shippingContactUpdate] = "shippingContactHandler"
        [tecPaymentRequestOptions] = "tecPrOptions"
        (canMakePaymentsResult)="handleCanMakePaymentsResult($event)"


    ></lib-tec-payment-request>
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
    isAppleUser: boolean = false;
    prObject: ApplePayPaymentRequest = examplePaymentRequestObject;

    tecPrOptions: TecPaymentRequestOptions = {
        applePayOptions: {
            buttonLanguage: 'ja',
            buttonType: 'book',
            buttonStyle: "white-outline"
        }
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
                const paymentResult: CompletePaymentResult = {
                    status: window['ApplePaySession'].STATUS_FAILURE,
                    errors: [
                        new window['ApplePayError']("shippingContactInvalid", "postalCode", "ZIP Code is invalid"),
                        new window['ApplePayError']("addressUnserviceable", "addressLines", "Cannot deliver to P.O. Box")
                    ]
                }
                
                completePayment(paymentResult);
            }
        }
        else {
            completePayment('success')
        }
    }

    cancelTrx: CancelListener = (cancelEvent: CancelEvent) => {
        console.log(cancelEvent);
    }

    shippingMethodHandler: ShippingMethodListener = (shippingMethodUpdateEvent: ShippingMethodEvent) => {
        const { shippingMethod, completeShippingMethodSelection } = shippingMethodUpdateEvent;
        const freeShipping: ApplePayShippingMethodUpdate = {
            newTotal: {
                label: "Item with Free Shipping",
                amount: "1.00",
                type: "final"
            }
        }

        const outOfCountryShipping: ApplePayShippingMethodUpdate = {
            newTotal: {
                label: "Item with Shipping Out of Network",
                amount: "101.00",
                type: "final"
            }
        }

        if (shippingMethod.identifier === "OutOfCountryShip")
            completeShippingMethodSelection(outOfCountryShipping);
        else
            completeShippingMethodSelection(freeShipping);
    }

    paymentMethodHandler: PaymentMethodListener = (paymentMethodEvent: PaymentMethodEvent) => {
        const { paymentMethod, completePaymentMethodSelection } = paymentMethodEvent;

                const examplePaymentUpdateObj: ApplePayPaymentMethodUpdate = {
                        newTotal: {
                            label: "Updated Payment Method",
                            amount: "0.00",
                            type: "final"
                        }
                    }

                if (paymentMethod.type === "credit") {
                    //Credit logic - if needed
                    const creditUpdateObj = { 
                        newTotal: {
                            ...examplePaymentUpdateObj['newTotal'],
                            amount: "1.10"
                        }
                    }

                    completePaymentMethodSelection(creditUpdateObj);
                }
                else if (paymentMethod.type === "debit") {
                    //Debit logic - if needed
                     const debitUpdateObj = { 
                        newTotal: {
                            ...examplePaymentUpdateObj['newTotal'],
                            amount: "0.10"
                        }
                    }

                    completePaymentMethodSelection(debitUpdateObj);
                }

                completePaymentMethodSelection(examplePaymentUpdateObj);
    }

    shippingContactHandler: ShippingContactListener = (shippingContactEvent: ShippingContactEvent) => {
        const { shippingContact, completeShippingContactSelection } = shippingContactEvent;

        const exampleShippingContactUpdateObj: ApplePayShippingContactUpdate = {
            newTotal: {
                label: "Test Positive Transaction for Shipping Update",
                amount: "1.00",
                type: "final"
            },
            newShippingMethods: [
                {    
                    label: 'Outside Shipping Network',
                    detail: "Arrives in 7-12 weeks",
                    amount: '100.00',
                    identifier: "OutOfCountryShip"
                },
                {    
                    label: 'Free Shipping',
                    detail: "Arrives in less than week",
                    amount: '0.00',
                    identifier: "FreeShipping"
                }
            ]
        }

        const errObj: ApplePayShippingContactUpdate = {
            newTotal: {
                label: "Test Negative Transaction for Shipping Update",
                amount: "1.00",
                type: "final"
            },
            errors: [
                {
                    errorType: "addressInvalid",
                    message: "Cannot Ship Outside the US"
                }
            ]
        }

        completeShippingContactSelection(
            (shippingContact.countryCode === "US") ? exampleShippingContactUpdateObj : errObj
        );
    }
}
```

# Apple Pay Error Handling
There are two [generic errors](#Generic-Errors) that can be safely built for Apple Pay callback functions, regardless of browsing agent.  However, if you know which browser the user is currently interacting with - you may also provide [Apple Pay specific errors](#Apple-Pay-Specific-Errors) that can provide a more detailed error experience for your users. The ```errors``` array will accept either [generic errors](#Generic-Errors) or [Apple Pay specific errors](#Apple-Pay-Specific-Errors), or a mixture of the two.  

## Generic Errors
Generic errors are an object that accept a ```"type"``` and ```"message"```.  There are currently two types:
- ```"addressInvalid"```
    - This error will notify user that the address they supplied (normally related to shipping address) is invalid for the transaction they chose. The message you supply can provide more details, if needed.
- ```"failure"```
    - This error will notify user with an "unknown" error and a message. Be aware that there are some instances (especially related to address info) in which the custom error message will not display correctly, and will only display an "unknown" error to the user. Since this option is difficult to direct the user how to correct the error - it's recommended to use this to catch errors which are actually unknown.
- Example:

```javascript
{
    type: "addressInvalid",
    message: "Shipping not available in selected country"
}
```
## Apple Pay Specific Errors
Apple Pay specific errors rely on proprietary JavaScript, that is (at the time of writing) available only in Safari. Beware that attempting to build an Apple Pay error on another browser (i.e. ```new ApplePayError``` on a Chrome browser) will result in errors.  

To that end - if you plan to provide Apple Pay specific errors - ensure the code only executes if Apple Pay is available. Here is an ApplePay example error below:

```javascript
import { Component } from '@angular/core';
import { CanMakePaymentsResult } from 'dist/te-connect-ng/lib/consts/paymentRequestTypeDefs';
import { 
    ApplePayPaymentRequest, 
    TecPaymentRequestError,
} from 'te-connect-ng';

import { examplePaymentRequestObject } from './consts';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
    isAppleUser: boolean = false;
    prObject: ApplePayPaymentRequest = examplePaymentRequestObject;
    

    handleCanMakePaymentsResult = (availablePrMethods: CanMakePaymentsResult) => {
        if (availablePrMethods && availablePrMethods.applePay) {
            //User is using Safari and has ApplePay capabilities (Wallet), with an active card loaded on their device.
            this.isAppleUser = true;

            const exampleAppleSpecificError: TecPaymentRequestError = new window['ApplePayError']("shippingContactInvalid", "postalCode", "ZIP Code is invalid");
        }
    }
}
```

# Apple Pay User Requirements
Once the TEConnect implementation steps for Apple Pay are followed, and an Apple Pay capable web app is being served (for both development and production) the Apple Button will only display when __all__ pre-requisites are fulfilled:
- Web Application pre-requisites:
    - Must be registered as a Magensa™ Token Exchange customer with Apple Pay capabilities.
        - This will grant you the ```appleMerchantId``` needed to supply to the ```teConnect``` instance.
        - Here is a link for [Magensa™ Support](https://magensa.net/support.html) - if needed.
    - Must register and verify public domain with Magensa™. Public domain must be served via ```https://``` only.
- End-User's (Browsing) Pre-requisites:
    - Must be using a compatible device:
        - Compatible iOS or macOS device with Apple Pay capabilities:
        -   [Apple's list of compatible device](https://support.apple.com/en-us/HT208531)
    - Must have an active card loaded into the "Wallet" of the compatible device.
    - Must be using Safari as the browsing agent. This may change in the future, but at the time of writing - Safari is the only approved browser for Apple Pay on the Web.
