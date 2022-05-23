<div align="center">
  <h1>Middy AWS event parse and normalization middleware</h1>
  <img alt="Middy logo" src="https://raw.githubusercontent.com/middyjs/middy/main/docs/img/middy-logo.svg"/>
  <p><strong>AWS event parsing and normalization middleware for the middy framework, the stylish Node.js middleware engine for AWS Lambda</strong></p>
<p>
  <a href="https://www.npmjs.com/package/@middy/event-normalizer?activeTab=versions">
    <img src="https://badge.fury.io/js/%40middy%2Fevent-normalizer.svg" alt="npm version" style="max-width:100%;">
  </a>
  <a href="https://packagephobia.com/result?p=@middy/event-normalizer">
    <img src="https://packagephobia.com/badge?p=@middy/event-normalizer" alt="npm install size" style="max-width:100%;">
  </a>
  <a href="https://github.com/middyjs/middy/actions/workflows/tests.yml">
    <img src="https://github.com/middyjs/middy/actions/workflows/tests.yml/badge.svg?branch=main&event=push" alt="GitHub Actions CI status badge" style="max-width:100%;">
  </a>
  <br/>
   <a href="https://standardjs.com/">
    <img src="https://img.shields.io/badge/code_style-standard-brightgreen.svg" alt="Standard Code Style"  style="max-width:100%;">
  </a>
  <a href="https://snyk.io/test/github/middyjs/middy">
    <img src="https://snyk.io/test/github/middyjs/middy/badge.svg" alt="Known Vulnerabilities" data-canonical-src="https://snyk.io/test/github/middyjs/middy" style="max-width:100%;">
  </a>
  <a href="https://lgtm.com/projects/g/middyjs/middy/context:javascript">
    <img src="https://img.shields.io/lgtm/grade/javascript/g/middyjs/middy.svg?logo=lgtm&logoWidth=18" alt="Language grade: JavaScript" style="max-width:100%;">
  </a>
  <a href="https://bestpractices.coreinfrastructure.org/projects/5280">
    <img src="https://bestpractices.coreinfrastructure.org/projects/5280/badge" alt="Core Infrastructure Initiative (CII) Best Practices"  style="max-width:100%;">
  </a>
  <br/>
  <a href="https://gitter.im/middyjs/Lobby">
    <img src="https://badges.gitter.im/gitterHQ/gitter.svg" alt="Chat on Gitter" style="max-width:100%;">
  </a>
  <a href="https://stackoverflow.com/questions/tagged/middy?sort=Newest&uqlId=35052">
    <img src="https://img.shields.io/badge/StackOverflow-[middy]-yellow" alt="Ask questions on StackOverflow" style="max-width:100%;">
  </a>
</p>
<p>You can read the documentation at: <a href="https://middy.js.org/docs/middlewares/event-normalizer">https://middy.js.org/docs/middlewares/event-normalizer</a></p>
</div>

Middleware for iterating through an AWS event records, parsing and normalizing nested events.

**AWS Events Transformations:**
https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html

Event Source       | Included | Comments
-------------------|----------|-----------------------------------------------
Alexa              | No       | Normalization not required
API Gateway (HTTP) | No *     | See middleware prefixed with `@middy/http-`
API Gateway (REST) | No *     | See middleware prefixed with `@middy/http-`
API Gateway (WS)   | No       | Opportunity for new middleware
Application LB     | No *     | See middleware prefixed with `@middy/http-`
CloudFormation     | No       | Normalization not required
CloudFront         | No       | Normalization not required
CloudTrail         | No       | Normalization not required
CloudWatch Logs    | Yes      | Base64 decode and JSON parse `data`
CodeCommit         | No       | Normalization not required
CodePipeline       | Yes      | JSON parse `UserParameters`
Cognito            | No       | Normalization not required
Config             | Yes      | JSON parse `invokingEvent` and `ruleParameters`
Connect            | No       | Normalization not required
DynamoDB           | Yes      | Unmarshall `Keys`, `OldImage`, and `NewImage`
EC2                | No       | Normalization not required
EventBridge        | No       | Normalization not required
IoT                | No       | Normalization not required
IoT Event          | No       | Normalization not required
Kafka              | Yes      | Base64 decode and JSON parse `value`
Kafka (MSK)        | Yes      | Base64 decode and JSON parse `value`
Kinesis Firehose   | Yes      | Base64 decode and JSON parse `data`
Kinesis Stream     | Yes      | Base64 decode and JSON parse `data`
Lex                | No       | Normalization not required
MQ                 | Yes      | Base64 decode and JSON parse `data`
RDS                | No       | Normalization not required
S3                 | Yes      | URI decode `key`
S3 Batch           | Yes      | URI decode `s3Key`
S3 Object Lambda   | No *     | See middleware `@middy/s3-object-response`
Secrets Manager    | No       | Normalization not required
SES                | No       | Normalization not required
SNS                | Yes      | JSON parse `Message`
SQS                | Yes      | JSON parse `body`

\* Handled in another dedicated middleware(s)

**Test Events**
Some events send test events after set, you will need to handle these.

```js
// S3 Test Event
{
  Service: 'Amazon S3',
  Event: 's3:TestEvent',
  Time: '2020-01-01T00:00:00.000Z',
  Bucket: 'bucket-name',
  RequestId: '***********',
  HostId: '***/***/***='
}
```

## Install

To install this middleware you can use NPM:

```bash
npm install --save @middy/event-normalizer
```

## Sample usage

```javascript
import middy from '@middy/core'
import eventNormalizer from '@middy/event-normalizer'

const lambdaHandler = (event, context) => {
  const { Records } = event
  for(const record of Records) {
    // ...
  }
}

const handler = middy(lambdaHandler).use(eventNormalizer())
```

## AWS SDK

This middleware depends on `aws-sdk` package. Lambda runtime already includes `aws-sdk` by default and as such you normally don't need to package it in your function. If you are using Webpack and Serverless Framework to package your code you'd want to add `aws-sdk` to Webpack `externals` and to `forceExclude` of Serverless Framework Webpack configuration.

1. Tell Webpack not to bundle `aws-sdk` into the output (e.g. handler.js):
```
# webpack.config.js

var nodeExternals = require("webpack-node-externals");

module.exports = {
  externals: ["aws-sdk"],
};
```

2. Tell Serverless Framework not to include `aws-sdk` located in `node_modules` (because it was not bundled by Webpack):
```
# serverless.yml

custom:
  webpack: # for webpack
    includeModules:
      forceExclude:
        - aws-sdk
  esbuild: # for esbuild
    exclude: ["aws-sdk"]
```

## Middy documentation and examples

For more documentation and examples, refers to the main [Middy monorepo on GitHub](https://github.com/middyjs/middy) or [Middy official website](https://middy.js.org).


## Contributing

Everyone is very welcome to contribute to this repository. Feel free to [raise issues](https://github.com/middyjs/middy/issues) or to [submit Pull Requests](https://github.com/middyjs/middy/pulls).


## License

Licensed under [MIT License](LICENSE). Copyright (c) 2017-2022 [Luciano Mammino](https://github.com/lmammino), [will Farrell](https://github.com/willfarrell), and the [Middy team](https://github.com/middyjs/middy/graphs/contributors).

<a href="https://app.fossa.io/projects/git%2Bgithub.com%2Fmiddyjs%2Fmiddy?ref=badge_large">
  <img src="https://app.fossa.io/api/projects/git%2Bgithub.com%2Fmiddyjs%2Fmiddy.svg?type=large" alt="FOSSA Status"  style="max-width:100%;">
</a>
