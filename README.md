
dawson documentation
====================

dawson is a [serverless](https://auth0.com/blog/what-is-serverless/) web framework for Node.js on AWS. dawson uses [AWS CloudFormation](https://aws.amazon.com/cloudformation/), [Amazon CloudFront](https://aws.amazon.com/cloudfront/), [Amazon API Gateway](https://aws.amazon.com/apigateway/) and  [AWS Lambda](https://aws.amazon.com/lambda/) to deploy the backend code and to manage the infrastructure for you. 

### Is dawson for me?
👍🏽 I'm building a️ single-page app/website with a backend  
👍🏽 I'm building an API-based service  
👍🏽 I'm building a server-rendered app/website  

The main goal of dawson is to be a zero-configuration yet fully extensible *[backend]* web framework for building web apps on AWS. You should be able to start using dawson without creating any configuration file and with only a basic knowledge of Amazon Web Services.

#### tl;dr show me the code!

```js
// api.js
export function greet (event) {
    const name = event.params.path.name
    return `Hello ${name}, you look awesome!`
}
greet.api = {
    path: 'greet/{name}'
}
```
```bash
$ npm install -g dawson
$ export AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... AWS_REGION=...
$ dawson deploy
```

# 0. Working with AWS

dawson requires Amazon Web Services credentials to operate. dawson needs the following environment variables:
- either `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_REGION`
- or `AWS_PROFILE` (with `AWS_REGION` if you're not using the profile's default region)

> **These credentials will be only used by dawson to create the CloudFormation Stack and to call sts.AssumeRole when using the *Development Server*. None of your app code will run with these credentials.**

### short version
Create an IAM user with `AdministratorAccess` permissions (be sure to create an Access Key), then create a profile with the given credentials (or export them as `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_REGION`).  

> Since we use the `aws-sdk-js`, any other method of setting credentials should work and can be used (e.g. EC2 Instance Role).

### long version for AWS beginners

1. create an Amazon Web Services Account or login into an existing account
2. from the top menu, choose Services and find *Identity & Access Management* (short: IAM)
3. choose Users from the left menu and click on Add User
4. enter an username (for example: *my-dawson-project*) and check the "Programmatic access" box. Click next.
5. click on "Attach existing policies directly" and search for a Policy named "AdministratorAccess", click next and confirm
6. the confirmation page will show a table with the credentials; you must write down the values of "Access key ID" (which usually starts with `AKIA`) and "Secret access key". Keep in mind that the value for Secret access key won't be shown again after you leave this page
7. point an AWS Region in which you want to deploy to; there's no default Region set. You may use `us-east-1` if you're located in the US or `eu-west-1` if you're located in EU. You need to choose a region in which AWS Lambda and Amazon API Gateway are supported, for more info check out the [AWS Region Table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).
7. from a terminal window your PC, which you'll use to run the dawson command, you may run:
```bash
export AWS_ACCESS_KEY_ID=...
export SECRET_ACCESS_KEY=...
export AWS_REGION=...
```

> You can find more information about AWS IAM Credentials here: https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys

> There are many other ways to set AWS Credentials on your PC, you may refer here for more info: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html 

# 1. Getting to know dawson

You write your app's code and then dawson takes care of building, packing, uploading the code to AWS and of creating the AWS infrastructure that your application needs to run.

### installing
you should install dawson using npm or yarn: `npm install -g dawson` or `yarn global add dawson`. You should then be able to run a `dawson --help` command.  
You're kindly invited to keep dawson up-to-date, starting with `v1.0.0` we will never introduce backwards-incompatible changes between non-major versions, following strict [SemVer](http://semver.org).

### working with *stage*s
You may want to have more than one deployment for your app, for example you might want to create separate *development* and *production* deployments: you can use the `--stage` parameter when running dawson (or set a DAWSON_STAGE env variable) to tell dawson which stage to operate on. By default, dawson uses a stage named `"default"`.
Different stages may also have different configurations, including different Domain Names.

### a notice about deployment speed
The *first deployment* will be very slow because many resoruces needs to be created (including a CloudFront distribution) and it will take anything between *15 to 45 minutes*. You can safely kill (Ctrl-C) the dawson command once it says "waiting for stack update to complete".
Subsequent deploys will *usually take around 2-5 minutes*, or more depending on the Resources that needs to be created and updated.

### under the hood
dawson reads the contents of a file named `api.js` in your current working directory. You should write (or just `export`) your functions in this `api.js` file.  

When you run the `$ dawson deploy` command, dawson reads your file's contents and constructs a description of the AWS infrastructure that needs to be created (functions, API endpoints, etc...). Such description is later fed to [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) which performs the actual deploy; the hard job of creating resources, calculating changes to deploy etc, is left to AWS. You might also opt-out using dawson anytime and simply edit the [CloudFormation Template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html) by yourself.


# 2. Working with functions

dawson deploys functions to the cloud and optionally makes them available via HTTP, if this statement looks weird to you, you may want to check out:
- https://www.quora.com/What-are-serverless-app (short)
- https://martinfowler.com/articles/serverless.html (long)
- https://aws.amazon.com/lambda/serverless-architectures-learn-more/ (PDF)
- https://docs.aws.amazon.com/lambda/latest/dg/lambda-introduction.html and https://aws.amazon.com/api-gateway/details/ (suggested readings)

Usually, a function, in dawson's terms, is an handler for an HTTP request *(much like a route in a koa/express app)*, which takes incoming parameters (such as HTTP Body, Querystring, etc) and returns an output to be displayed in a browser.

You should place all of your functions in a file named `api.js` (or you might define them elsewhere and just `export`). The `api.js` file will be parsed and automatically transpiled using `babel` so you can use any JavaScript language feature that's supported by [`babel-preset-env`](https://github.com/babel/babel-preset-env), including ES6 Modules, ES7 `Array.prototype.includes` etc.

Each function in the `api.js` file **must** have an `api` property, which tells dawson some information about your function's behaviour; more on this in the *Configuring functions* chapter.

All the lines logged by your functions (via `console.log`, `console.error`, `process.stdout.write` etc...) will be automatically delivered to [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html), a persistent and searchable Log Storage. Logs can be later fetched or streamed using `$ dawson logs`.

#### Basic function: HTML
```js
// an helloWorld function will be created which, when invoked,
// will return an HTML string
export function helloWorld (event) {
    console.log('this function was called with this parameter:', event);
    return `
        <html>
            <body>
                <h1>hello world</h1>
            </body>
        </html>
    `;
}
helloWorld.api = {
    path: 'hello', // the http path to attach this function to
    method: 'GET'  // & the http method
};
```

#### Basic function: JSON
```js
// an helloWorld function will be created which, when invoked,
// will return a JSON string. The returned object will be automatically
// serialized using JSON.stringify(...)
export function helloWorld (event) {
    console.log('this function was called with this parameter:', event);
    return { hello: 'world' };
}
helloWorld.api = {
    path: 'hello', // the http path to attach this function to
    method: 'GET', // & the http method (defaults to GET)
    responseContentType: 'application/json'
                   // & the Content-type (defaults to 'text/html')
};
```

## 2.1 Function programming model

### 2.1.1 Parameters

Given a generic function definition:
```js
function helloWorld (event, context) { /* ... */ }
helloWorld.api = {
    path: 'xyz'
};
```

If `api.path === false`, the `event` parameter will be exactly the event Object that the Lambda receives from other AWS services.

If `api.path !== false` the `event` parameter will be an Object with the following properties:
```js
{
  "params": {
    "querystring": {}, // parameters from the querystring
    "path": {}, // path parmeters, captured by `{}` in function `path`s
    "header": {} // some HTTP headers from the client, see below
  },
   // body: the request body (useful only for POST and PUT requests)
   //  currently, only application/json and application/www-form-urlencoded bodies will be parsed
  "body": Object | string,
  "meta": {
    // expectedResponseContentType: the content-type that is expected to be returned
    //  by the function. This is used internally to wrap the returned value
    //  for the API Gateway Method Response.
    "expectedResponseContentType": "the value from api.responseContentType property"
  },
  "stageVariables" : {
    // API Gateway's Stage Variables if you set any of them,
    // empty by default
  }
}
```

The second parameter, `context`, is [Lambda's Context](https://docs.aws.amazon.com/lambda/latest/dg/programming-model-v2.html). You should rarely need to access this property. **Do not call** ~~`context.done`~~, ~~`context.fail`~~ or ~~`context.succeed`~~.


### 2.1.2 Return value

A function can return:
* a `string`, which will be returned as-is as the HTTP response;
* an `Object`, which will be JSON.stringified and returned as the HTTP response (note that returning an object makes sense only if `api.responseContentType === "application/json"`);
* a `Promise`, which fulfills with any of the previous types; **you can also declare your function as `async` and use `await` in it!**

Currently, functions can not modify HTTP Response Headers.

### 2.1.3 Returning an HTTP Redirect

To respond with an HTTP Redirect (with an HTTP Status equal to `307 Temporary Redirect`), you must return an Object with a `Location` property. Additionally, the `api.redirects` configuration property must also be set to `true`.

```js
function myRoute () {
    // your logic, including await etc...
    return { Location: 'http://www.google.com' };
}
myRoute.api = {
    redirects: true
};
```

> Due to limitations in API Gateway, you cannot return any payload and you cannot mix redirecting and non-redirecting responses.

### 2.1.4 Responding with an Error

When a function fails and an error occurs, you can throw an `Error` or return a rejecting `Promise`.
dawson hides all uncaught Errors and will not leak any information about it. The Client (either a browser or any other HTTP agent) will receive a generic `HTTP 500 Internal Server Error`.

You may obviously want to return expected (i.e. *Handled*) errors to the client, instead.
dawson supports custom error responses, using the following model:

Instead of 
~~```throw new Error('I wanted to throw a 403 error')```~~ you should write:
```js
throw new Error(JSON.stringify({
  httpStatus: 403, // int
  response: 'I am throwing a 403 error' // string
}));
// Note that the Error constructor accepts just a String as first argument,
// so you need to use JSON.stringify.
```

When returning an Error like such, the specified `httpStatus` HTTP Status Code will be set on the Response and:

* if `responseContentType === "application/json"`, the whole error payload is JSON.stringified and returned as the HTTP response
* in all other cases, the `response` property is returned as the HTTP response

Currently, dawson supports the following httpStatus codes: `400`, `403`, `404`, `500` (using other Status Codes will result in an API Gateway Internal Error).



asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd
asdasdasd

