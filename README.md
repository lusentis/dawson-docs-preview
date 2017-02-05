
dawson documentation
====================

dawson is a [serverless](https://auth0.com/blog/what-is-serverless/) web framework for Node.js on AWS ([CloudFormation](https://aws.amazon.com/cloudformation/), [CloudFront](https://aws.amazon.com/cloudfront/), [API Gateway](https://aws.amazon.com/apigateway/), [Lambda](https://aws.amazon.com/lambda/)).  
You can use `dawson` to build and deploy backend code and infrastructure for single-page apps + API, pure APIs or server-rendered pages.

The main goal of dawson is to be a zero-configuration yet fully extensible web framework for building web apps on AWS. You should be able to start using dawson without creating any configuration file and with only a basic knowledge of Amazon Web Services.

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

