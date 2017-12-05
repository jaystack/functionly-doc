# Deployment
This article describes how you can deploy the Functionly application and what will happen during the deployment. 

* [Functionly CLI](#functionly-cli)
    * [Package creation](#package-creation)
    * [Deploy application](#deploy-application)
* [Stage](#stage)
* [Supported deployment targets](#supported-deployment-targets)
    * [Local](#local-deployment)
    * [AWS](#aws-deployment)
* [Plugins](#plugins)



## Functionly CLI
[[Functionly CLI|cli]] is the tool managing the deployment process. It helps you with package creation and with the deployment of the application. There are multiple supported deployment targets, like [local](#local-deployment) or [AWS](aws-deployment) but you can also create your own deployment target by creating [functionly cli plugin](#plugins).

When you use Functionly in your project or after installing it to global, you already have Functionly CLI.
```sh
./node_modules/.bin/functionly --version
```

# Package creation
Environment deployment implemenatations can support package creation before the deployment to see the deployment resources.
```sh
functionly package [target]
```
> `[target]` is optional. When it is not set, input is coming from `functionly.json` which contains default values. It throws an error when the default values are missing.

**AWS** deployment is also supported. When you create a package for an AWS environment, you can see the created resources under the `dist/aws` folder.

# Deploy application
You can deploy the application into different environments. The deploy command usually creates a new package using the current source.
```sh
functionly deploy [target]
```
> `[target]` is optional. When it is not set, input is coming from `functionly.json` which contains default values. It throws an error when the default values are missing.

# Stage
You can use different stages for your application. You can set the current stage in the relevant CLI with the `--stage <name>` commandline argument. If unset, the value falls back to default value that is defined in `functionly.json`. If even this isn't set, then the default is `dev`.

This is used to mark your resources to know what resources are used in which stage of the application. For example, in an **AWS** environment the Dynamo tables, S3 storages, etc... will be suffixed with the name of the stage.
> This means that the `name of the resource` will be different from the one you defined using a specific value of the decorator

DynamoTable example in the `dev` stage:
```js
@dynamoTable({ tableName: 'Users-table' })
class Users extends DynamoTable {}
```
The name of the DynamoTable will be `Users-table-dev` in AWS. Also, Functionly will know the stage when running and use it to access the proper table during runtime.
```js
export class CreateUser extends Service {
    public async handle(@param userData, @inject(Users) users: Users) {
        await users.put({ Item: userData })
    }
}
```
The injected instance of the User is also aware of the current stage and the proper Dynamo table.

# Supported deployment targets
Currently, there are two supported deployment targets in Functionly. The Local environment and the AWS platform.

## Local deployment
By default, in local environment Functionly uses local DynamoDB. You need a DynamoDB in your local environment for this (for example [this](https://hub.docker.com/r/peopleperhour/dynamodb/)). The local deployment creates the tables in the DynamoDB if they do not exist.
> The `package` command of CLI is not supported. It directly checks the DynamoDB.

## AWS deployment
The deployment of the application in an AWS environment is done via [CloudFormation](https://aws.amazon.com/cloudformation/). Packaging creates CloudFormation templates and these will be used to create and update the application resources (IAM roles, Lambdas, Dynamo tables, S3 storages, etc...)

# Plugins
With plugins of the CLI you can create new deployment targets or change / extend the behaviors of the existing ones.

[This article]() descibe how the plugins work in Functionly. Examples can be found [here](https://github.com/jaystack/functionly-plugin-examples).
