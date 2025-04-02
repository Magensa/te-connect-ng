# TEConnect Angular Component
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-ng.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-ng "@magensa/te-connect-ng npm.js")  

Angular component for use with Token Exchange Connect (also referred to as TEConnect).  
TypeScript Definitions are exposed, and can be imported from this library as well.  

## `te-connect-ng` v2 (Angular ^19)
This `main` branch details implementation instructions for `te-connect-ng` v2, which has migrated to `"standalone"` components, and requires Angular ^19. All `te-connect-ng` v1 versions use NgModules. Documentation for v1 can be found in this repo, on the legacy `master` branch.

# Getting Started
Include Token Exchange Connect (core logic) and Token Exchange Connect Angular library in your workspace.
```
npm install @magensa/te-connect @magensa/te-connect-ng
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-ng
```
## Magensa™
TEConnect Manual Entry, TEConnect 3DS Manual Entry, and [TEConnect Payment Request](./TecPaymentRequestREADME.md) components all require a valid [Magensa™](https://magensa.net/) account. If you need assistance creating or configuring an account, please reach out to the [Magensa Support Team](https://magensa.net/support.html).  

## Card Manual Entry Component
This document will cover the card Manual Entry component that TEConnect offers - as well as the optional [3DS Manual Entry component](#TecThreeDs-3DS-Component).  

TEConnect also offers a [Payment Request](./TecPaymentRequestREADME.md) component, with both Apple Pay and Google Pay supported. [Payment Request Documentation can be found here](./TecPaymentRequestREADME.md)  

Below we will begin with a step-by-step integration of the card manual entry component. Continue reading for the optional 3DS manual entry component. Be aware that only one manual entry component may be present on the DOM at a time.  When utilizing both components for different use-cases, ensure there is a condition in place that determines which manual entry to mount.  

If you would prefer to let the code speak, there are [example implementations](#Example-Implementations).

# Step-By-Step  
1. The first step is to create a ```TEConnect``` instance with your public key (provided by Magensa&trade;). Feed that instance to the ```TeConnectNgComponent``` using the input named `[teConnect]`:  

`app.component.ts`  

```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectNgComponent, TEConnect } from '@magensa/te-connect-ng';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

@Component({
  selector: 'app-root',
  imports: [ TeConnectNgComponent ],
  template: `
    <lib-te-connect-ng 
      #teconnectng
      [teConnect] = "tecInstance"
    ></lib-te-connect-ng>
  `,
  styleUrl: './app.component.css'
})
export class AppModule { 
  tecInstance = TE_CONNECT;
}
```  

2. Next, populate the library markup with inputs of your choosing. For a minimally viable product - add a click handler to the markup, and a function of your own to capture the response of the ```createPayment``` method:  
```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect';
import { 
  TeConnectNgComponent, 
  TEConnect, 
  CreatePaymentResponse 
} from '@magensa/te-connect-ng';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

@Component({
  selector: 'app-root',
  imports: [ TeConnectNgComponent ],
  template: `
    <lib-te-connect-ng 
      #teconnectng
      [teConnect] = "tecInstance"
      (teResponse) = "handleTecResponse($event)"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment()">
      Create Payment Token
    </button>
  `,
  styleUrl: './app.component.css'
})
export class AppModule { 
  tecInstance = TE_CONNECT;

  handleTecResponse( response: CreatePaymentResponse ): void {
    const { error } = response;
    
    if (error)
      console.log("Unsuccessful, message reads: ", error);
    else
      console.log('result:', response);
  }
}
```  

The above is a minimal example, and will get you started right away. There are additional configuration options, such as:  
  - [Custom Styles configuration](#Styles-API)
  - [3DS Manual Entry](#TecThreeDs-3DS-Component)
  - [Optional Billing ZIP Code](#Example-Implementation-NoZip)

# Styles API
The optional styles object (```StylesConfig```) is used to customize the manual entry inputs to your specifications.  You may also style the outer container itself, using CSS, by targeting the id: `"__te-connect-secure-window"`.  
The object is composed of two main properties:
- ```base```  
    - General styles applied to the container.
- ```boxes```
    - Styles applied to the input elements.

Below we have the complete API with examples of default values for each.

### Base
| Property Name | Parent Property | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|:---:|
| backgroundColor | base | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | container background color |
| margin | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container margin |
| padding | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container padding |
| direction | wrapper | ```string``` | ```'row', 'row-reverse', 'column', 'column-reverse'``` | ```'row'``` | ```'flex-direction'``` style property |
| flexWrap | wrapper | ```string``` | ```'wrap', 'wrap', 'wrap-reverse'``` | ```'wrap'``` | ```'flex-wrap'``` style property |
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes | 
| inputSize | variants | ```string``` | ```"small", "medium"``` | ```"medium"``` | input component size |  
| autoMinHeight | variants | ```boolean``` | ```boolean``` | ```false``` | ```true``` will maintain a static margin on each input box that will not grow with validation errors | 
  
<br />

### Default Base Object:
```javascript
{
    base: {
        wrapper: {
            margin: '1rem',
            padding: '1rem',
            direction: 'row',
            flexWrap: 'wrap'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'normal',
            inputSize: 'medium',
            autoMinHeight: false
        },
        backgroundColor: '#fff'
    }
}
```
<br />

### Boxes
| Property Name | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|
| labelColor | ```string``` | jss color (rgb, #, or color name) | ```"#3f51b5"``` | label text and input outline (or underline) color |
| textColor | ```string``` | jss color (rgb, #, or color name) | ```"rgba(0, 0, 0, 0.87)"``` | color of text for input value *Also applies :onHover color to outline/underline* |
| borderRadius | ```number``` | numerical unit for css ```border-radius``` property | 4 | border radius for input boxes |
| inputColor | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | input box background color |  
| errorColor | ```string``` | jss color (rgb, #, or color name) | ```"#f44336"``` | Error text and box outline (or underline) color |  
  
<br />

### Default Boxes Object:
```javascript
{
    boxes: {
        labelColor: "#3f51b5",
        textColor: "rgba(0, 0, 0, 0.87)",
        borderRadius: 4,
        errorColor: "#f44336",
        inputColor: "#fff"
    }
}
```

# TEConnect Options
The second parameter of the ```createTEConnect``` method is an options object. This object is optional.
| Property Name  | Input Type | Notes |
|:--:|:--:|:--:|
| billingZip | ```boolean``` | See [Optional Zip Code Implementation](#Example-Implementation-NoZip) |
| tecPaymentRequest | ```TecPaymentRequestOptions``` | See the [Payment Request README for more info](./TecPaymentRequestREADME.md) |
| threeds | `TecThreeDsOptions` | See [TecThreeDs Component](#TecThreeDs-3DS-Component) for more info |


```typescript
type TecPaymentRequestOptions = {
    appleMerchantId?: string,
    googleMerchantId?: string
}

type TecThreeDsOptions = {
    threedsApiKey: string,
    threedsEnvironment: "sandbox" | "production"
}

type CreateTEConnectOptions = {
    hideZip?: boolean,
    tecPaymentRequest?: TecPaymentRequestOptions,
    threeds?: TecThreeDsOptions
}
```

# TecThreeDs (3DS) Component
3DS Manual Entry is an optional feature.  Implementation requires configuring your TEConnect instance to allow 3DS manual entry (detailed below), and use of the `TecThreedsComponent`.  Be aware that only one manual entry component may be present on the DOM at a time.  When utilizing both components for different use-cases, ensure there is a condition in place that determines which manual entry to mount to the DOM.  

To configure your TEConnect instance for 3DS Manual Entry (opt-in), add a `threeds` parameter to the [the ```options``` object](#TEConnect-Options) for the `createTEConnect` method.  
Add a 3DS API key (`threedsApiKey`) to the `threeds` object, and (optionally) a `threedsEnvironment` string as well (defaults to `sandbox` for testing, when not specified). Only flip your project to `production` when you have tested with `sandbox`, and are ready to deploy to production.  

Note that `'sandbox'` environment requires the use of test cards. [3DS `sandbox` Test Cards can be found here](https://docs.3dsintegrator.com/docs/test-cards#emv-3ds-test-cards).

### _Opt In_
```javascript
    const teInstance = createTEConnect("__publicKeyGoesHere__", { 
        threeds: {
            threedsApiKey: "__3dsApiKeyGoesHere__",
            threedsEnvironment: "sandbox"
        }
    });
```  

3DS workflow is used in a similar manner as manual entry. A required [`threeDsConfigObject`](#ThreeDsConfigObject) must be fed to the `TecThreedsComponent`, via the `[threeDsConfig]` property.  For a minimally viable 3DS solution, that is the only additional requirement needed to complete the 3DS manual entry workflow. There is also an optional `[threeDsStatus]` listener, which can provide more context on the 3DS progression during progress.

See the [TecThreeDs Example](#TecThreeDsExample) for an example implementation.

## `ThreeDsConfigObject`
TEConnect forwards the `threeDsConfigObject` to 3DS API call 'authenticate browser'. This is the underlying method begins the 3DS process; TEConnect handles these interactions on your application's behalf while the cardholder is entering the card info. TEConnect only requires two properties to get started: 
- `challengeNodeId: string` - the id of a node that must be mounted on the DOM. This node is used to mount a "challenge" iframe. 
    - The challenge iframe is defined by the issuer of the card being authenticated (in the event that a "challenge" is required to complete the 3DS authentication). This challenge varies based upon the issuer of the card. [See ThreeDsChallenge for more info](#ThreeDs-Challenge)
- `amount: number` - the final amount for the transaction. Required for 3DS Auth. 

Please note: There are four properties, documented in the [3DS API documentation](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), that TEConnect handles and will be ignored if supplied in the ThreeDsConfigObject: `browser` object, and card info (`pan`, `year`, and `month`).

There are many optionally properties available, in addition to the required `challengeNodeId` and `amount` properties. 3DS has extensive documentation on the properties available in this call, which [can be found here](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser).  With the exception of the `browser` object, `pan`, `year`, and `month` - all other options supplied to the `TecThreedsComponent` will be forwarded to the 3DS API call.


## ThreeDs Status Listener
### `[threeDsStatus]`
Subscribe to this event to listen for status updates with the 3DS workflow.  
See the [TecThreeDs Example](#TecThreeDsExample) for an example how to subscribe.

```typescript
type ThreeDsMethods = "GENERATE_JWT" | "AUTHENTICATE_BROWSER" | "FINGERPRINT_DEVICE" | "CHALLENGE";
type ThreeDsStatus = "REQUESTED" | "SUCCESS" | "FAIL" | "AWAIT_RESULTS";

type TecThreeDsStatus = {
    threedsMethod: ThreeDsMethods,
    status: ThreeDsStatus,
    message: string
}

exampleStatusListener(statusEvent: TecThreeDsStatus) : void {
  console.log(statusEvent);
}
```
```html
  <lib-tec-threeds
      #tecThreeDs
      (teResponse) = "handleTecResponse($event)"
      [teConnect] = "tecInstance"
      [threeDsConfig] = "tecThreeDsConfig"
      [threeDsStatus] = "exampleStatusListener"
  ></lib-tec-threeds>
```

## ThreeDs Challenge
There are many possible 3DS responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)).  When a status of type `"C"` is returned - this indicates the card issuer has requested the cardholder to complete a challenge, in order to proceed. In this case - TEConnect will build the challenge iframe, and mount it on the node with the id provided (via `challengeNodeId`).  

The particular challenge rendered varies greatly depending on card issuer. The cardholder must complete the challenge in order to proceed with 3DS Authentication.

There are a few options available in the [ThreeDsConfigObject](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), in regards to a challenge.  Such as `challengeIndicator` to specify if a challenge is able to be presented to the cardholder.  Other options include `challengeWindowSize` and `transactionForcedTimeout` (if an early timeout is desired).  

TEConnect will attempt to collect the challenge results 10 seconds after a challenge is rendered. If the results are not yet available, it will poll until results are available or timeout. It will timeout after 60 seconds.

Since the challenge look and feel will vary depending on issuer, as well as the operations needed to complete the challenge - it's recommended to mount the challenge on a node that is styled with `absolute` positioning.  If you wish to place the challenge in a modal - make use of the [`threeds-status`](#ThreeDs-Status-Listeners) and listen for the status: `{ threedsMethod: "CHALLENGE", status: "REQUESTED" }` to trigger the modal.  

## Additional ThreeDs Considerations
3DS can have many potential responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)). While 3DS auth may or may not be successful, or approved - please note that a token is always returned with a successful response, and can be used to call MPPG's `ProcessToken` for processing.  It's up to the merchant whether or not to assume the risk in the transaction. You may want to be familiar with responses, as they relate to various card networks, to determine if a liability shift has occured.  For example: [Visa](https://docs.3dsintegrator.com/docs/visa), [MasterCard](https://docs.3dsintegrator.com/docs/mastercard#faud-liability-shift-conditions), etc.  

When `"sandbox"` environment is in use - the `threeDSRequestorURL` will use a placeholder value of `"https://your.domainname.com"`. When flipped to `"production"` - the origin of your web application will be used.  This is because `http:` and `localhost` domains fail validation for 3DS calls, but can be useful for early development.  Be aware that your `"production"` web application must be deployed using a valid `https://` domain, for 3DS to be successful.

<br />  

# createPayment Return Objects
These are the possible objects that will be returned *successfully* from the ```createPayment``` function. Thrown errors will be thrown as any other async method.  
  
  1. ### Success:
```typescript
{
    magTranID: string,
    timestamp: string,
    customerTranRef: string,
    token: string,
    code: string,
    message: string,
    status: number,
    cardMetaData: null | {
      maskedPAN: string,
      expirationDate: string,
      billingZip: null | string
    },
    threedsResults?: ThreeDsResults
}
```  
  
  2. ### Bad Request
```typescript
{
    magTranID: string,
    timestamp: string,
    customerTranRef: string,
    token: null,
    code: string,
    message: string,
    error:  string,
    cardMetaData: null
}
```

3. ### Error (Failed Validation, Timeout, Mixed Protocol, etc)
```typescript
{ error: string }
```

4. ### ThreeDsResults (property of Success response, if 3DS opt-in)  
    All 3DS responses documented in the [3DS Documentation](https://docs.3dsintegrator.com/reference/subscribetoupdates)  

```typescript
type ThreeDsResults = {
    scaRequired?: bool,
    creq?: string,
    status?: string,
    authenticationValue?: string,
    authenticationType?: string,
    eci?: string,
    acsUrl?: string,
    dsTransId?: string,
    acsTransId?: string,
    sdkTransId?: string,
    error?: string,
    errorDetail?: string,
    errorCode?: string,
    cardholderInfo?: string,
    transactionId?: string,
    correlationId?: string,
    code?: number
}
```

# Example Implementations
Basic implementation, with custom styles below.  

```app.component.ts```
```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect';
import { 
  TeConnectNgComponent, 
  TEConnect, 
  CreatePaymentResponse, 
  StylesConfig
} from '@magensa/te-connect-ng';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

@Component({
  selector: 'app-root',
  imports: [ TeConnectNgComponent ],
  template: `
    <lib-te-connect-ng 
      #teconnectng
      [teConnect] = "tecInstance"
      (teResponse) = "handleTecResponse($event)"
      [stylesConfig] = "myStyles"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment()">
      Create Payment Token
    </button>
  `,
  styleUrl: './app.component.css'
})
export class AppModule { 
  tecInstance = TE_CONNECT;

  handleTecResponse( response: CreatePaymentResponse ): void {
    const { error } = response;
    
    if (error) {
      /* 
        Unsuccessful - see message for reason.
        Be aware - there are cases when a response object will return in addition to an error message.
        If this is the case - it will include a magTranId and customerTranRef that can be useful
        if the issue persists. In the case of an error with object, the token will be set to 'null'
      */
      console.log("Unsuccessful, message reads: ", error);
    }
    else {
      //Success
      console.log('result:', response);
    }
  }

  //Custom Styles are optional.
  myStyles: StylesConfig = {
    base: {
        wrapper: { 
            margin: 0, 
            padding: 0
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'dense'
        },
        backgroundColor: 'rgb(66, 66, 66)'
    },
    boxes: {
      labelColor: "#BB86FC",
      textColor: "#BB86FC",
      borderRadius: 10,
      errorColor: "#CF6679",
      inputColor: '#121212'
    }
  }
}
```
  
# Example Implementation NoZip  
You may choose to hide the "ZIP Code" input field upon creating a TEConnect instance. Once the field is hidden - ```billingZip``` becomes an optional parameter to the ```createPayment``` function.  If you would still like the Payment Token created to include a billingZip - you may supply your own value (as demonstrated below). You may also choose to omit the zip code completely, and the payment token will be created without a billing ZIP code.  

```app.module.ts```  
```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect';
import { 
  TeConnectNgComponent, 
  TEConnect, 
  CreatePaymentResponse, 
  StylesConfig
} from '@magensa/te-connect-ng';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

@Component({
  selector: 'app-root',
  imports: [ TeConnectNgComponent ],
  template: `
    <lib-te-connect-ng 
      #teconnectng
      [teConnect] = "tecInstance"
      (teResponse) = "handleTecResponse($event)"
      [stylesConfig] = "myStyles"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment(customerZip)">
      Create Payment Token
    </button>
  `,
  styleUrl: './app.component.css'
})
export class AppModule { 
  tecInstance = TE_CONNECT;

  //You may optionally supply a billing ZIP Code if desired - otherwise call the function without a parameter.
  customerZip = "90025";

  handleTecResponse( response: CreatePaymentResponse ): void {
    const { error } = response;
    
    if (error) {
      /* 
        Unsuccessful - see message for reason.
        Be aware - there are cases when a response object will return in addition to an error message.
        If this is the case - it will include a magTranId and customerTranRef that can be useful
        if the issue persists. In the case of an error with object, the token will be set to 'null'
      */
      console.log("Unsuccessful, message reads: ", error);
    }
    else {
      //Success
      console.log('result:', response);
    }
  }

  //Custom Styles are optional.
  myStyles: StylesConfig = {
    base: {
        wrapper: { 
            margin: 0, 
            padding: 0
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'dense'
        },
        backgroundColor: 'rgb(66, 66, 66)'
    },
    boxes: {
      labelColor: "#BB86FC",
      textColor: "#BB86FC",
      borderRadius: 10,
      errorColor: "#CF6679",
      inputColor: '#121212'
    }
  }
}
```

//TODO:
# TecThreeds (3DS) Manual Entry Example 
TEConnect 3DS Manual Entry example below.  If using both Manual Entry and 3DS Manual Entry - ensure there is a condition in place that will only render one manual entry form (`<lib-te-connect-ng` or `lib-tec-threeds`).

```app.module.ts```  
```typescript
import { Component } from '@angular/core';
import { createTEConnect } from '@magensa/te-connect';
import { 
  TecThreedsComponent, 
  TEConnect, 
  CreatePaymentResponse, 
  StylesConfig,
  ThreeDsConfigObj,
  TecThreeDsStatus
} from '@magensa/te-connect-ng';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");
const exampleThreedsConfig: ThreeDsConfigObj = {
    amount: 110,
    challengeNodeId: "example-challenge-node" /* ensure the node with the corresponding id is mounted on the DOM. Likely in a parent component. */
};


@Component({
  selector: 'app-root',
  imports: [ TecThreedsComponent ],
  template: `
    <lib-tec-threeds
        #tecThreeDs
        (teResponse) = "handleTecResponse($event)"
        [teConnect] = "tecInstance"
        [stylesConfig] = "myStyles"
        [threeDsConfig] = "tecThreeDsConfig"
        [threeDsStatus] = "exampleStatusListener"
    ></lib-tec-threeds>

    <button type="button" (click)="tecThreeDs.createPayment()">
      Create Payment Token
    </button>
  `,
  styleUrl: './app.component.css'
})
export class AppModule { 
  tecInstance = TE_CONNECT;
  tecThreeDsConfig = exampleThreedsConfig;

  exampleStatusListener(statusEvent: TecThreeDsStatus) : void {
    console.log(statusEvent);
  }

  handleTecResponse( response: CreatePaymentResponse ): void {
    const { error } = response;
    
    if (error) {
      /* 
        Unsuccessful - see message for reason.
        Be aware - there are cases when a response object will return in addition to an error message.
        If this is the case - it will include a magTranId and customerTranRef that can be useful
        if the issue persists. In the case of an error with object, the token will be set to 'null'
      */
      console.log("Unsuccessful, message reads: ", error);
    }
    else {
      //Success
      console.log('result:', response);
    }
  }

  //Custom Styles are optional.
  myStyles: StylesConfig = {
    base: {
        wrapper: { 
            margin: 0, 
            padding: 0
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'dense'
        },
        backgroundColor: 'rgb(66, 66, 66)'
    },
    boxes: {
      labelColor: "#BB86FC",
      textColor: "#BB86FC",
      borderRadius: 10,
      errorColor: "#CF6679",
      inputColor: '#121212'
    }
  }
}
```

