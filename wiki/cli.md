# Functionly CLI
Functionly is not just a module, it is also a CLI. It helps you in packaging, deploying and running your serverless application.

* [Install](#install)
* [CLI configuration](#cli-configuration)
* [Project configuration](#project-configuration)
* [Commands](#commands)
* [Plugins](#plugins)

# Install
You have to install cli before use. There is two way to use, first when you install it to global, then you can use it in any project.
```sh
npm install -g functionly
```
Or install it for the project.
```sh
npm install --save functionly
```
# CLI configuration
## for AWS
When you want to deploy the application to aws cloud, you have to configure the aws credential on the machine. The configuration steppes and possibilities are same as when you want to use AWS CLI. You can read more [here](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) about the possibilities.

# Project configuration
Just create `functionly.json` in the project root and you can set defaults for the project. When you set these then you don't have to set them as CLI commandline option if you don't want to override them.
- **awsRegion**: string
    
    The target aws region to host the application
- **awsBucket**: string

    The deployment bucket name of the application
    
    default: auto generated from name of the project
- **main**: string

    The entry point of the application
- **deployTarget**: string

    default deploy target
- **localPort**: number

    The default port where the express app listen in the local run
- **stage**: string

    The default stage of the application. The application can have more stages. Read more [here]() about stages
    
    default: dev
- **debug**: boolean

    debug mode
    
    default: false

Example `functionly.json `
```js
{
    "awsRegion": "eu-central-1",
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
Or use the help of the CLI, it is also works for the commands.
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
With package command you can create the deployment package of the application for the destination environment. With this, you can see the deployment resources.
> Not all providers support this command.

> Deployment usualy create a new package before deployment, any changes in the package usualy does not apply at deployment.
```sh
functionly package
```
## Deploy command
You can deploy the application into different environments. Deploy command usualy create a new package from the current source.
This section has an own page [[here|deployment]]
```sh
functionly deploy
```
## Local command
Possibility to run the application in the local machine is important in development time because the deployment process can more then acceptable. With this command you can host the application in your machine via express.
```sh
functionly local
```

## Serverless command
> It does not support every functionly features!

Write your application in functionly and you can create the serverless.yml from it. And then you can use the [serverless cli](https://github.com/serverless/serverless)
```sh
functionly serverless
```

# Plugins
You have possibility to create plugins to Functionly CLI. The plugin can extend the available commands, change / extend command behavior or define new deployment targets. [This article]() descibe how the plugins work in Functionly. Examples can be found [here](https://github.com/jaystack/functionly-plugin-examples).

