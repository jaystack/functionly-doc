# Classes
Available classes in functionly

## Functionly core classes
- [Resource](#resource)
- [FunctionalService](#functionalservice)
- [Service](#service)
- [Api](#api)
- [Hook](#hook)
- [PreHook](#prehook)
- [PostHook](#posthook)

## AWS related classes
- [DocumentClientApi](#documentclientapi)
- [DynamoTable](#dynamotable)
- [S3Api](#s3api)
- [S3Storage](#s3storage)
- [SNSApi](#snsapi)
- [SimpleNotificationService](#simplenotificationservice)
- [ApiGateway](#apigateway)


# Resource
This is a base class of every resource what functionly can use. Its handle the environment variables propagation to the [FunctionalServices](#functionalservice)

## FunctionalService
FunctionalServices are the entrypoints of the applications. These can subscribe to the event sources when annotated with [rest]() or [eventSource]() decorators. The services have to implement a public `handle` method what contains a business logic. These can inject other resources ([Api](), [Service]() or an other [FunctionalService]()) or collect parameters ([param](), [stage](), etc...)
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    public async handle(@param name) {
        return `hello ${name}`
    }
}
```
> You have to export the `Home.createInvoker()` result from the `main.js` to publish the [FunctionalService]()

### Inject another FunctionalService
You can inject a FunctionalService from another. In AWS its a cross lambda call, in local its a request to the other FunctionalService.
```js
@rest({ path: '/helloworld' })
class About extends FunctionalService {
    public async handle(@param name, @inject(Home) home) {
        const homeResult = await home.invoke({ name })
        return { homeResult }
    }
}
```
!!! known issue: For the local environment: you have to decorate the injected [FunctionalService]() with the [rest]() decorator because its called over the first exposed endpoint.

## Service
The Services helps to organize the code what you want reuse with [inject](). Services are stateless but if you need states then use [Api](). The services have to implement a public `handle` method. These services can only resolve an invoke parameters with [param]() decorator so they can't read for example from request body.

### Definition
```js
@injectable()
class Greeter extends Service {
    public async handle(@param name) {
        return `hello ${name}`
    }
}
```
### Usage
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    public async handle(@param name, @inject(Greeter) greeter) {
        return await greeter({ name })
    }
}
```

## Api
The Apis helps to organize the code what you want reuse with [inject](). Apis can store states but they can **disapear** any time because of serverless technology. In these classes you can inject other resources in the `constructor`. These has `init` function what is an async method and invoked and waited before the api is injected.
Any other function on these classes are simple javascript functions.

### Definition
```js
@injectable()
class NameValidator extends Service {
    public async handle(@param name) {
        return name === 'John'
    }
}
@injectable()
class GreeterApi extends Api {
    public constructor(@inject(NameValidator) private nameValidator) { }
    public async init() { }

    public async hello(name) {
        if (await this.nameValidator(name))
            return `hello ${name}`
        return `hello anonymous`
    }
    public async bye(name) {
        return `bye ${name}`
    }
}
```
### Usage
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    public async handle(@param name, @inject(GreeterApi) greeter: GreeterApi) {
        return await greeter.hello(name)
    }
}
```


## Hook
Hook is a base class of hooks. If use it as base class then it is works like [PreHook](#prehook).

## PreHook
Hooks are middlewares in functionly you can chain them with the [use]() decodator on [FunctionalServices](#functionalservice) or an other [Hook](#hook). PreHooks are running `before` the decorated class. With the `@use(ResolveUser)` decoration means the `ResolveUser` hook bound to the `Home` FunctionalService. And the `@inject(ResolveUser) user` parameter in the `Home`'s handle method can recive the result.
```js
@injectable()
class ResolveUser extends PreHook {
    public async handle(@param name, @param age) {
        return { name, age }
    }
}

@rest({ path: '/helloworld' })
@use(ResolveUser)
class Home extends FunctionalService {
    public async handle(@param name, @inject(ResolveUser) user) {
        console.log(user)
    }
}
```


## PostHook
Hooks are middlewares in functionly you can chain them with the [use]() decodator on [FunctionalServices](#functionalservice) or an other [Hook](#hook). PostHooks are running `after` the decorated class. With the `@use(TransformResult)` decoration means the `TransformResult` hook bound to the `Home` FunctionalService.
```js
class TransformResult extends PostHook {
    public async handle(@result result) {
        return { result }
    }

    public async catch(@error error) {
        return { error }
    }
}

@rest({ path: '/helloworld' })
@use(TransformResult)
class Home extends FunctionalService {
    public async handle() {
        return [1, 2, 3]
    }
}
```
When there is no error the result is:
```js
{
    result: [1, 2, 3]
}
```
And when error occured:
```js
{
    error: errorValue
}
```


## DocumentClientApi
DocumentClientApi creates the `AWS.DynamoDB.DocumentClient` connection to the [DynamoTable](#dynamotable).
> In local environment its connect to `http://localhost:8000`. What you can change when you set the `DYNAMODB_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## DynamoTable
When you define a `DynamoTable` and its related to any published [FunctionalService]() the deployment will create a DynamoDB table for the application. And you can [inject]() the `Users` table to use it like an [Api](). There are functions on it to work with records.
```js
@dynamoTable({ tableName: '%ClassName%-table'})
class Users extends DynamoTable {}
```


## S3Api
S3Api creates the `AWS.S3` connection to [S3Storage](#s3storage).
> In local environment its connect to `http://localhost:4572`. What you can change when you set the `S3_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## S3Storage
When you define a `S3Storage` and its related to any published [FunctionalService]() the deployment will create a S3 Bucket for the application. And you can [inject]() the `Files` storage to use it like an [Api](). There are functions on it to work with files.
```js
@s3Storage({ bucketName: '%ClassName%-store'})
class Files extends S3Storage {}
```


## SNSApi
S3Api creates the `AWS.SNS` connection to [SimpleNotificationService](#simplenotificationservice).
> In local environment its connect to `http://localhost:4100`. What you can change when you set the `SNS_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## SimpleNotificationService
When you define a `SimpleNotificationService` and its related to any published [FunctionalService]() the deployment will create a Simple Notification Service topic for the application. And you can [inject]() the `Notifications` storage to use it like an [Api](). There is function on it to publish message.
```js
@sns({ topicName: '%ClassName%-topic'})
class Notifications extends SimpleNotificationService {}
```
> On every deploy there is a new topic created because of the cloudformation can't update custom named topics. The created topic name will contains an additional random number.


## ApiGateway
It is just a technical class for [eventSource]() decorator. This allows you to subscribe to ApiGateway events like a DynamoTable record or S3 file changes. Its not work in local environment as like [apiGateway]().