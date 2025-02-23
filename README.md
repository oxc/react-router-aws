# react-router-aws

## AWS adapters for React Router v7 (successor to Remix)

<div align="center">
  <p align="left">
    <a href="https://www.npmjs.com/package/react-router-aws?activeTab=versions">
      <img src="https://badge.fury.io/js/react-router-aws.svg" alt="npm version" style="max-width:100%;">
    </a>
    <a href="https://packagephobia.com/result?p=react-router-aws">
      <img src="https://packagephobia.com/badge?p=react-router-aws" alt="npm install size" style="max-width:100%;">
    </a>
    <a href="https://snyk.io/test/github/oxc/react-router-aws">
      <img src="https://snyk.io/test/github/oxc/react-router-aws/badge.svg" alt="Known Vulnerabilities" data-canonical-src="https://snyk.io/test/github/oxc/react-router-aws" style="max-width:100%;">
    </a>
  </p>
</div>

Forked from [remix-aws](https://github.com/wingleung/remix-aws) to support React Router v7, which Remix was merged into.

## 🚀 support

- API gateway v1
- API gateway v2
- Application load balancer

## Getting started

```shell
npm install --save react-router-aws
```

```javascript
// server.js
import * as build from '@react-router/dev/server-build'
import {AWSProxy, createRequestHandler} from 'react-router-aws'

export const handler = createRequestHandler({
    build,
    mode: process.env.NODE_ENV,
    awsProxy: AWSProxy.APIGatewayV2
})
```

### `awsProxy`

By default the `awsProxy` is set to `AWSProxy.APIGatewayV2`.

#### Options

- `AWSProxy.APIGatewayV1`
- `AWSProxy.APIGatewayV2`
- `AWSProxy.ALB`
- `AWSProxy.FunctionURL`

## Vite preset

If you use Vite, then the `awsPreset` preset is an easy way to configure aws support.
It will do a post React Router build and create a handler function for use in aws lambda.

There is no need for a separate `server.js` file. The preset will take care of that.
However, if you want to manage your own `server.js` file, you can pas a custom `entryPoint` to your own `server.js`.

⚠️ By default React Router will set `serverModuleFormat` to `esm`.
The Vite preset will automatically align the `serverModuleFormat` with the esbuild configuration used by the preset.
However, to ensure that AWS lambda correctly interprets the output file as an ES module, you need to take additional steps.

There are two primary methods to achieve this:

- Specify the module type in package.json:
  Add `"type": "module"` to your package.json file and ensure that this file is included in the deployment package sent to AWS Lambda.

- Use the .mjs extension:
  Alternatively, you can change the file extension to `.mjs`. For example, you can configure the React Router `serverBuildFile` setting to output `index.mjs`.

more info: [AWS docs on ES module support in AWS lambdas](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html#designate-es-module)

```typescript
import type { PluginOption } from 'vite'
import type { Preset } from '@remix-run/dev'
import { reactRouter } from "@react-router/dev/vite";

import { awsPreset } from 'react-router-aws/vite';
import { AWSProxy } from 'react-router-aws';
import { defineConfig } from 'vite'

export default defineConfig(
  {
    // ...
    plugins: [
      reactRouter({
        // serverBuildFile: 'index.mjs', // set the extension to .mjs or ship you package.json along with the build package
        presets: [
          awsPreset({
            awsProxy: AWSProxy.APIGatewayV2,
    
            // additional esbuild configuration
            build: {
              minify: true,
              treeShaking: true,
              ...
            }
          }) as Preset
        ]
      }) as PluginOption,
    ]
  }
)
```

**Example [server.js](./templates/server.js)**

```typescript
import { AWSProxy, createRequestHandler } from 'react-router-aws'

let build = require('./build/server/index.js')

export const handler = createRequestHandler({
  build,
  mode: process.env.NODE_ENV,
  awsProxy: AWSProxy.APIGatewayV1
})
```


### configuration

#### `awsProxy` is optional and defaults to `AWSProxy.APIGatewayV2`

#### `build` is for additional esbuild configuration for the post React Router build

```json
// default esbuild configuration
{
  logLevel: 'info',
  entryPoints: [
    'build/server.js'
  ],
  bundle: true,
  sourcemap: false,
  platform: 'node',
  outfile: 'build/server/index.js', // will replace react router server build file
  allowOverwrite: true,
  write: true,
}
```
check [esbuild options](https://esbuild.github.io/api/#build-options) for more information

## Notes

### split from @remix/architect

As mentioned in [#3173](https://github.com/remix-run/remix/pull/3173) the goal would be to provide an AWS adapter for
the community by the community.
In doing so the focus will be on AWS integrations and less on Architect. I do think it's added value to provide examples
for Architect, AWS SAM, AWS CDK, Serverless,...

**info:** [ALB types](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/aws-lambda/trigger/alb.d.ts#L29-L48)
vs [API gateway v1 types](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/aws-lambda/trigger/api-gateway-proxy.d.ts#L116-L145)
