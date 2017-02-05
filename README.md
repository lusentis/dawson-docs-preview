
dawson documentation
====================

dawson is a [serverless](https://auth0.com/blog/what-is-serverless/) web framework for Node.js on AWS ([CloudFormation](https://aws.amazon.com/cloudformation/), [CloudFront](https://aws.amazon.com/cloudfront/), [API Gateway](https://aws.amazon.com/apigateway/), [Lambda](https://aws.amazon.com/lambda/)).  
You can use `dawson` to build and deploy backend code and infrastructure for single-page apps + API, pure APIs or server-rendered pages.

The main goal of dawson is to be a zero-configuration yet fully extensible web framework for building web apps on AWS. You should be able to start using dawson without creating any configuration file and with only a basic knowledge of Amazon Web Services.

#### Basic usage

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
7. from a terminal window your PC, which you'll use to run the dawson command, you may run run:
```bash
export AWS_ACCESS_KEY_ID=...
export SECRET_ACCESS_KEY=...
export AWS_REGION=...
```

You can find more information about AWS IAM Credentials here: https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys

There are many other ways to set AWS Credentials on your PC, you may refer here for more info: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html 

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

