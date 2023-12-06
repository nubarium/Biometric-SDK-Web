# Biometric SDK for WEB
Nubarium Biometrics WEB SDK guides for developers.

## SDK compatibility

- Android - Chrome (Version 104 and above)
- Windows - Chrome (Version 56 and above) & Edge
- iOS -  Safari  (Version 14.8 and above)
- MacOS - Safari & Chrome (Version 56 and above)

## Components

The library includes 3 components,

- FaceCapture
- IdCapture
- VideoSelfie

## Installation

### Prerequisites

- You must have your Nubarium credentials. 

### Library

**Step 1: Include the library and it dependencies**

```html
<!-- Third party dependency bundle -->
<script src="//cdn.nubarium.com/nubSdk/nubSdk@latest/nubSdk-third.min.js"></script>
<!-- Library -->
<script src="//cdn.nubarium.com/nubSdk/nubSdk@latest/nubSdk-biometrics.min.js"></script>
```

## Integrate

### Before you begin

- Get the Nubarium Key or API Credentials. It is required to successfully initialize the SDK.
- Implement the REST API call in your back-end to generate the [JWT token](https://github.com/nubarium/-Biometric-SDK-Web/blob/main/JWT.md), the documentation is also available on a Postamn Collection (https://documenter.getpostman.com/view/23758200/2s83zgvR3A#faa41449-0a17-4634-826a-774566d5bedb).
- All the steps in this document are mandatory unless stated otherwise.

### Reference

- Rest API SDK Documentation.

### Versions

- Rest API SDK Documentation.

 

## Initializing the SDK

### Face Capture

#### **Step 1: Initialize the SDK**

**HTML Element**

It is necessary to first declare the HTML Element where the component where the component will be drawn.

```html
<div id="face_component"></div>
```

Then you will need to declare the javascript code that calls the library

```javascript
// Class initialization with the Application Context
let faceCapture = new FaceCapture();

// Define your configuration
let config = {
  rootElement: 'face_component',   // DOM Element that will contains the HTML Component
  maxValidations: 3,
  features: {   // Optional
    disabled: ["glasses", "facemask"], //Default values // Does not allow glasses and facemask
    enabled: []
  },
  antispoofing: {     // Default values
    enabled: true,   
    level: 3
  }
};
// Initialize the component with your custom configuration
faceCapture.init(config);
// Set the JWT token
faceCapture.setToken('eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Im51Y.....'); // YOUR JWT TOKEN
```

##### Configuration options

- **rootElement** (req): DOM Element where the component will be drawn.
- **maxValidations** (req): Max number of failed evaluations, the default values is 3.
- **features** (opt): Specify the features that are enabled or disabled, by default the use of `glasses` and `facemark` are disabled.
- **antispoofing** (opt): The anti spoofing is enabled by default, but you can swith to just face capture.
- **ui** (opt) : The color of the screen messages can be customized, by default are white with a black stroke.
- **timeout** (opt): Elapsed time in miliseconds to finish the test. (Default value of `180000`) 
- **cameras** (opt): Specify the cameras allowed to perform the test in order to show to the user, it can be an array list of options or a string, the accepted values are 'front', 'back', 'default'.  Example, `['front', 'back', 'default']`

#### Step 2: Load the component

Before to start the component you should load the component , in case of any error the event onError will be called.

```javascript
faceCapture.load(()=>{
  // Once that the component is loaded then it will execute the following
  // If you requires to start the component automatically after load its recommend it to start the component.
  faceCapture.start();
}); 
```

#### Step 3: Setting a event listener for results and load

To receive the images and result of component execution it is necessary to setting up a result listener.

```javascript
faceCapture.onEvaluation((evaluation, result) => {
  let ev = evaluation; // success, fail
  // The result object is similar to the onSuccess or onFail.
});
faceCapture.onSuccess((data) => {
  let id = data.id;
  let result = data.result;
  let face = data.face;
  let frame = data.frame;  
}).onFail((fail) => {
  let id = fail.id;  
  let reason = fail.reason;
  let result = fail.result;
}).onError((error) => {
  let errorCode = error.code;
  let errorMsg = error.msg;
});
```

##### Output

- **OnSuccess**

  - *id*: Transaction id, 
  - *result*: Object with the result of the evaluation.
    - score: Score index of the evaluation
    - evaluation: Result of evaluation (`pass` , `warning`)
    - retro: List with retro tags  (blurred, very_blurry, glasses, facemask)   . *(Experimental)*
  - *face*: Face image represented as Base64 string.
  - *frame*: Frame image represented as Base64 string.

- **OnFail**

  - *reason*: Reason of the fail (   `capture_face_timeout` , `low_evaluation`, `no_face`,`face_outside` , `facemask_not_allowed`, `glasses_not_allowed` )

  - *result*: if the reason capture_low_evaluation, the result is and object with the score. 

    - score: Score index of the evaluation.
    - retro: List with retro tags  (`blurred`, `very_blurry`, `id_attack`, `glasses`, `facemask`)   . *(Experimental)*

    

#### Step 4: Start component

We recommen to start the component right after the load,  but in case it is required to start the component after specific action you can execute the following command.

```javascript
faceCapture.start();
```

#### Other methods

- clear() : Finish all connections and removes the elements drawed from DOM.
- retry() : Retry the test without remove the DOM elements.
- getVersion() :  Get the string values of the component version.

### IdCapture

#### **Step 1: Initialize the SDK**

**HTML Element**

It is necessary to first declare the HTML Element where the component where the component will be drawn.

```html
<div id="id_component"></div>
```

Then you will need to declare the javascript code that calls the library

```javascript
// Class initialization with the Application Context
let idCapture = new IdCapture();

// Define your configuration
let config = {
  rootElement: 'id_component',   // DOM Element that will contains the HTML Component
  timeouts: {
    front: 180000,
    back: 180000
  },
  captureMode: {
    front:{
      enabled: true,
      after: 7500
    },
    back:{
      enabled: true,
      after: 7500
    }    
  },
  // Enable/Disable image guide
  guide: {
  	front: { 
    	enabled: true //Enable guide on front capture. (By default is enabled)
    },
    back: {
    	enabled: true,  //Enable guide on back capture. (By default is enabled)
      until: 10000   // wait for 10 seconds to hide the image
    }
  },
  autorotate: true, //Automatic rotate the image to return a landscape image.
  antispoofing: {     // Default values
    enabled: true,   
    level: 3
  }  
};
// Initialize the component with your custom configuration
idCapture.init(config);
// Set the JWT token
idCapture.setToken('eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Im51Y.....'); // YOUR JWT TOKEN
```

##### Configuration options

- **rootElement** (required): DOM Element where the component will be drawn.
- **timeouts** (optional): Elapsed time in miliseconds to finish the test per side. (Default value of `180000`) 
- **captureMode** (optional): Define whether you desire to disable or enable the capture mode after n milliseconds.
- **guide** (optional): Defines whether you want to enable the credential guide image to help the end user capture it. By default it will be enabled but you can disable it if desired or indicate the time after which it is disabled.
- **autorotate** (optional): Flag to force auto rotate (orientation) of the output image for front and back, by default it is disabled. It is recommended to enable it since the OCR reading can give better results
- **antispoofing** (optional): Configuration to enable or disable credential validation or vary the level of strict to handle.

#### Step 2: Load the component

Before to start the component you should load the component , in case of any error the event onError will be called.

```javascript
idCapture.load(()=>{
  // Once that the component is loaded then it will execute the following
  // If you requires to start the component automatically after load its recommend it to start the component.
  idCapture.start();
}); 
```

#### Step 3: Setting a event listener for results and load

To receive the images and result of component execution it is necessary to setting up a result listener.

```javascript
idCapture.onSuccess((data) => {
  let id = data.id;
  let front = data.front;
  let back = data.back;  
  let result = data.result;
}).onFail((fail) => {
  let id = data.id;  
  let reason = fail.reason;
  let result = fail.result;
}).onError((error) => {
  let errorCode = error.code;
  let errorMsg = error.msg;
});
```

##### Output

**OnSuccess**
- *id*: Transaction id, 
- *result*: Object with the result of the evaluation.
  - score: Score index of the evaluation
  - evaluation: Result of evaluation (`pass` , `warning`)
  - retro: List with retro tags  (`high_accurate_ocr`, `ocr_not_readable`, `medium_accurate_ocr`) 
- *front*: Front ID image represented as Base64 string.
- *back*: Back ID image represented as Base64 string.
- *mode*: Object with the capture mode.
  - front: Capture mode (`passive` , `active`).
  - back: Capture mode (`passive` , `active`).
- *document*: Object with the document classification.
  - class: Main classification of the document (`INE` , `IFE`).
  - subclass: Capture mode (`C` , `D`, `EF`, `GH`).

*ocr*: Object with the *ocr*  retroalimentation.

- label: Labels identified in the OCR validation.
  - Frente : *MEX, INE_TIT, IFE_TIT, CRED, REGISTRO, NOMBRE, EDAD, SEXO, NACIMIENTO, DOMICILIO, CVE_ELECTOR, CURP, ESTADO, SECCION, ANO_REG, LOCALIDAD, VIG_HASTA, VIGENCIA, EMISION, MUNICIPIO, NACIMIENTO_VAL, VIGENCIA_VAL, ANO_REG_VAL, CVE_ELECTOR_VAL, CURP_VAL*
  - Reverso: *IDMEX*

**OnFail**

- *reason*: Reason of the fail ( `capture_front_timeout`   ,  `capture_back_timeout`   ,`low_evaluation`  , `ocr_not_readable`, `low_accurate_ocr` )
- *result*: if the reason capture_low_evaluation, the result is and object with the score. 
  - score: Score index of the evaluation.
  - retro: List with retro tags  (`high_accurate_ocr`, `ocr_not_readable`, `low_accurate_ocr`, `medium_accurate_ocr`)   . *(Experimental)*



#### Step 4: Start component

We recommen to start the component right after the load,  but in case it is required to start the component after specific action you can execute the following command.

```javascript
idCapture.start();
```

#### Other methods

- clear() : Finish all connections and removes the elements drawed from DOM.
- retry() : Retry the test without remove the DOM elements.
- getVersion() :  Get the string values of the component version.

### Video Selfie

#### **Step 1: Initialize the SDK**

**HTML Element**

It is necessary to first declare the HTML Element where the component where the component will be drawn.

```html
<div id="video_component"></div>
```

Then you will need to declare the javascript code that calls the library.

**Example with a voice recording**

```javascript
// Class initialization with the Application Context
let videoRecorder = new VideoRecorder();

// Define your configuration
let config = {
  rootElement: 'video_component',   // DOM Element that will contains the HTML Component
  textToPronounce: 'Yo Juan perez acepto la solicitud de credito',
  timeouts: {
    face: 90000,
    front: 120000,
    back: 120000
  },
  guide: {  // Enable/Disable image guide
  	front: { 
    	enabled: true  //Enable guide on front capture. (By default is enabled)
    },
    back: {
    	enabled: true,  //Enable guide on back capture. (By default is enabled)
      until: 10000   // wait for 10 seconds to hide the image
    }
  }
};
// Initialize the component with your custom configuration
videoRecorder.init(config);
// Set the JWT token
videoRecorder.setToken('eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Im51Y.....'); // YOUR JWT TOKEN
```

**Example without a voice recording**

```javascript
// Class initialization with the Application Context
let videoRecorder = new VideoRecorder();

// Define your configuration
let config = {
  rootElement: 'video_component',   // DOM Element that will contains the HTML Component
  timeouts: {
    face: 90000,
    front: 120000,
    back: 120000
  },
  guide: {  // Enable/Disable image guide
  	front: { 
    	enabled: true  //Enable guide on front capture. (By default is enabled)
    },
    back: {
    	enabled: true,  //Enable guide on back capture. (By default is enabled)
      until: 10000   // wait for 10 seconds to hide the image
    }
  },
  tasks: {
  	disabled: ["transcript"]
  }
};
// Initialize the component with your custom configuration
videoRecorder.init(config);
// Set the JWT token
videoRecorder.setToken('eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Im51Y.....'); // YOUR JWT TOKEN
```

##### Configuration options

- **rootElement** (required): DOM Element where the component will be drawn.
- **textToPronounce** (optional): Text to be pronounced if the end user is required to mention a text and validate that he pronounces it correctly, for this purpose the option can also be configured with ***limitScore***.
- **limitScore** (optional): Value that can be specified if and only if the option  ***textToPronounce*** is used. 
- **timeouts** (optional): Elapsed time in miliseconds to finish the test per side. (Default value of `180000`) 
- **captureMode** (optional): Define whether you desire to disable or enable the capture mode after n milliseconds.
- **guide** (optional): Defines whether you want to enable the credential guide image to help the end user capture it. By default it will be enabled but you can disable it if desired or indicate the time after which it is disabled..
- **tasks** (optional): You can enable which tasks to enable or disable, to do this you can specify the disabled attribute with the following available values:`face_capture`, `id_capture_front`, `id_capture_back` o `transcript`.

#### Step 2: Load the component

Before to start the component you should load the component , in case of any error the event onError will be called.

```javascript
videoRecord.load(()=>{
  // Once that the component is loaded then it will execute the following
  // If you requires to start the component automatically after load its recommend it to start the component.
  videoRecord.start();
}); 
```

#### Step 3: Setting a event listener for results and load

To receive the images and result of component execution it is necessary to setting up a result listener.

```javascript
videoRecord.onSuccess((data) => {
  let id = data.id;
  let result = data.result;
  // The following is available just for audio recording.
  let similarity = result.similarity;
  let difference = result.difference;
}).onFail((fail) => {
  let id = data.id;  
  let reason = fail.reason;
}).onError((error) => {
  let errorCode = error.code;
  let errorMsg = error.msg;
});
```

#### Step 4: Start component

We recommen to start the component right after the load,  but in case it is required to start the component after specific action you can execute the following command.

```javascript
videoRecord.start();
```
