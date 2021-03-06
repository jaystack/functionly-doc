# Functionly CLI
Functionly is not only an application, it is also a CLI. It helps you in packaging, deploying and running your serverless applications.

* [Install](#install)
* [CLI configuration](#cli-configuration)
* [Project configuration](#project-configuration)
* [Commands](#commands)
* [Plugins](#plugins)

# Install
First, you have to install CLI. You can either install it to global, or you can install it to use it in any project.
```sh
npm install -g functionly
```
Or install it for the project.
```sh
npm install --save functionly
```
# CLI configuration
## for AWS
When you want to deploy the application to the AWS cloud, you have to configure AWS credentials. The configuration steps and options are the same as those you want to use with AWS CLI. You can read more [here](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

# Project configuration
Just create `functionly.json` in the project root and you can set the defaults for the project. After setting these, you don't have to set them as CLI commandline options, unless you want to override them.
- **awsRegion**: string
    
    The target AWS region to host the application
- **awsBucket**: string

    The deployment Bucket name of the application
    
    default: auto generated from name of the project
- **main**: string

    The entry point of the application
- **deployTarget**: string

    default deploy target
- **localPort**: number

    The default port where the express app is listening
- **stage**: string

    The default stage of the application. The application can have multiple stages. Read more [here]() about stages
    
    default: dev
- **debug**: boolean

    debug mode
    
    default: false

Example `functionly.json `
```js
{
    "awsRegion": "us-east-1",
    "main": "./lib/main.js",
    "deployTarget": "aws",
    "localPort": 3000,
    "stage": "dev"
}
```

# Commands
You can check the current version of the CLI with the following command
```sh
functionly --version
```
You can use the internal help of the CLI, which also works for the commands.
```sh
functionly --help
functionly <command> --help
```
List of commands
* [package](#package-command)
* [deploy](#deploy-command)
* [local](#local-command)
* [serverless](#serverless-command)

## Package command
With the package command you can create the deployment package of the application for the target environment. You can also see the deployment resources with this.
> Not all providers support this command.

> Package creation will onl create the package which will be deployed, but if the immediate next step is deployment, then this won't be the package which is deployed as the deployment process will create its own package. Therefore, any modification on this package will be lost!


```sh
functionly package
```
## Deploy command
You can deploy the application into different environments. Deploy command usually creates a new package from the current source.
This section has an own page [[here|deployment]]
```sh
functionly deploy
```
## Local command
The option to run the application in a local environment is important during development because the deployment process can be more error-free this way. With this command you can host the application in your local environment using Express.
```sh
functionly local
```

## Serverless command
> It does not support all the Functionly features!

Write your application in Functionly and you can create the serverless.yml from it. Then you can use the [serverless cli](https://github.com/serverless/serverless)
```sh
functionly serverless
```

# Plugins
You have option to create plugins to Functionly CLI. Plugins can extend the available commands, change / extend command behaviors or define new deployment targets. [This article]() describes how the plugins work in Functionly. Examples can be found [here](https://github.com/jaystack/functionly-plugin-examples).

