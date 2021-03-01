# TEConnect Angular Component
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-ng.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-ng "@magensa/te-connect-ng npm.js")  

Angular component for use with Token Exchange Connect utility.  
TypeScript Type Defs are exposed, and can be imported from this module as well.  

# Getting Started
```
npm install @magensa/te-connect @magensa/te-connect-ng
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-ng
```

If you would prefer to let the code speak, below we have an [example implementation](#-Example-Implementation)

1. The first step is to create a ```TEConnect``` instance with your public key. Feed that instance into the ```TEConnectModuleNg``` where you are importing it:  
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect'

import { AppComponent } from './app.component';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

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

2. Next, in your component, define the library markup, and a handler to receive the response of the ```createPayment``` method:  
```typescript
import { Component } from '@angular/core';
import { CreatePaymentResponse } from '@magensa/te-connect-ng';

@Component({
  selector: 'app-root',
  template: `
    <lib-te-connect-ng 
        #teconnectng
        (teResponse) = "handleTEResponse($event)"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment()">Create Payment Token</button>  
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'te-connect-example';

  handleTEResponse( response: CreatePaymentResponse ): void {
    console.log(response);
}
```  

3. The above is a minimal example, and will get you started right away. There are additional configuration options, such as:  
  - [Custom Styles configuration](#-Styles-API)
  - [Optional Billing ZIP Code](#-Example-Implementation-NoZip)


# createPayment Return Objects
These are the possible objects that will be returned *successfully* from the ```createPayment``` function. Thrown errors will be thrown as any other async method.  
  
  1. ### Success:
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: String,
    code: String,
    message: String
    status: Number
}
```  
  
  2. ### Bad Request
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: null,
    code: String,
    message: String,
    error: String
}
```

3. ### Error (Failed Validation, Timeout, Mixed Protocol, etc)
```typescript
{ error: String }
```

# Styles API
The styles object (```type: StylesConfig```) injected is composed of two main properties:
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
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes |  
  
<br />

### Default Base Object:
```javascript
{
    base: {
        wrapper: {
            margin: '1rem',
            padding: '1rem'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'normal'
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

# Example Implementation
```app.module.ts```
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect'

import { AppComponent } from './app.component';

const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__");

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
import { CreatePaymentResponse, StylesConfig } from '@magensa/te-connect-ng';

@Component({
  selector: 'app-root',
  template: `
    <lib-te-connect-ng 
        #teconnectng
        (teResponse) = "handleTEResponse($event)"
        [stylesConfig] = "myStyles"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment()">Create Payment Token</button>  
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'te-connect-example';

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

  handleTEResponse( response: CreatePaymentResponse ): void {
    console.log(response);
    const { error } = response;
    if (error) {
      /* 
        Unsuccessful - see message for reason.
        Be aware - there are cases when a response object will return in addition to an error message.
        If this is the case - it will include a magTranId and customerTranRef that can be useful
        if the issue persists. In the case of an error with object, the token will be set to 'null'
      */
    }
    else {
      //Success
    }
  }
}
```

  
# Example Implementation NoZip  
You may choose to hide the "ZIP Code" input field upon creating a TEConnect instance. Once the field is hidden - ```billingZip``` becomes an optional parameter to the ```createPayment``` function.  If you would still like the Payment Token created to include a billingZip - you may supply your own value (as demonstrated below). You may also choose to omit the zip code completely, and the payment token will be created without a billing ZIP code.  

```app.module.ts```  
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { TeConnectNgModule } from '@magensa/te-connect-ng';
import { createTEConnect } from '@magensa/te-connect'

import { AppComponent } from './app.component';

//The form will not contain a ZIP Code input field with this option.
const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", { hideZip: true });

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
import { CreatePaymentResponse, StylesConfig } from '@magensa/te-connect-ng';

@Component({
  selector: 'app-root',
  template: `
    <lib-te-connect-ng 
        #teconnectng
        (teResponse) = "handleTEResponse($event)"
        [stylesConfig] = "myStyles"
    ></lib-te-connect-ng>

    <button type="button" (click)="teconnectng.createPayment(customerZip)">Create Payment Token</button>  
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'te-connect-example';

  //You may optionally supply a billing ZIP Code if desired - otherwise simply call the function without a parameter.
  customerZip = "90025";

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

  handleTEResponse( response: CreatePaymentResponse ): void {
    console.log(response);
    const { error } = response;
    if (error) {
      /* 
        Unsuccessful - see message for reason.
        Be aware - there are cases when a response object will return in addition to an error message.
        If this is the case - it will include a magTranId and customerTranRef that can be useful
        if the issue persists. In the case of an error with object, the token will be set to 'null'
      */
    }
    else {
      //Success
    }
  }
}
``` 
