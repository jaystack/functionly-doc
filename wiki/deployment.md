# Deployment
This article describes how can you deploy the functionly application and what happen druring the deployment. 

* [Functionly CLI](#functionly-cli)
    * [Package creation](#package-creation)
    * [Deploy application](#deploy-application)
* [Stage](#stage)
* [Supported deployment targets](#supported-deployment-targets)
    * [Local](#local-deployment)
    * [AWS](#aws-deployment)
* [Plugins](#plugins)



## Functionly CLI
[[Functionly CLI|cli]] is a tool for the deployment, it helps you in package creation and deployment of the application. There are more deployment targets are implemented, like [local](#local-deployment) or [AWS](aws-deployment) but there is a possibility to create own target by creating [functionly cli plugin](#plugins).

When you use functionly in the project, then already you have Functionly CLI (or after install it to global)
```sh
./node_modules/.bin/functionly --version
```

# Package creation
Environment deployment implemenatation can support the package creation before the the deployment to see the deployment resources.
```sh
functionly package [target]
```
> The `[target]` is optional. When the target not set it comes from the `functionly.json` what is contain the default values. But throws an error when not set at all.

**AWS** deployment support it. When you create package for aws environment, you can see the created resources in the `dist/aws` folder.

# Deploy application
You can deploy the application into different environments. Deploy command usualy create a new package from the current source.
```sh
functionly deploy [target]
```
> The `[target]` is optional. When the target not set it comes from the `functionly.json` what is contain the default values. But throws an error when not set at all.

You can deploy the application into different environments. Deploy command usualy create a new package from the current source.

# Stage
You can use different stages from the application. You can set the current stage in the current cli command with the `--stage <name>` commandline argument. But when you not set the value fallbacks to default value what is defined in `functionly.json` or when not set at all the value has a default what is `dev`.

This is used for mark your resources to know what resources used for what stage of the application. For example, in **AWS** environment the Dynamo tables, S3 storage, etc... will be suffixed with the name of the stage.
> This means the `name of the resource will be different` what you defined with the specific value of the decorator

DynamoTable example in the `dev` stage:
```js
@dynamoTable({ tableName: 'Users-table' })
class Users extends DynamoTable {}
```
The name of the DynamoTable will be `Users-table-dev` in AWS. Also the functionly will know the stage in runtime and use it to access the proper table in runtime.
```js
export class CreateUser extends Service {
    public async handle(@param userData, @inject(Users) users: Users) {
        await users.put({ Item: userData })
    }
}
```
The injected instance of the User knows the current stage and the proper Dynamo table.

# Supported deployment targets
Currently there are two supported deployment target in Functionly. The Local and the AWS platform.

## Local deployment
By default, in local environment the functionly uses local DynamoDB. You need a DynamoDB in local (for example [this](https://hub.docker.com/r/peopleperhour/dynamodb/)). The local deployment creates the tables in the DynamoDB if it not exists.
> The `package` command of CLI is not supported. It is directly check the DynamoDB.

## AWS deployment
The deployment of the application in AWS environmet is operate via [CloudFormation](https://aws.amazon.com/cloudformation/). The packaging creates CloudFormation templates and these will be used to create and update the application resources (IAM roles, Lambdas, Dynamo tables, S3 storages, etc...)

# Plugins
With plugins of the CLI you can create new deployment targets or change / extend behavior of the implemented one.

[This article]() descibe how the plugins work in Functionly. Examples can be found [here](https://github.com/jaystack/functionly-plugin-examples).