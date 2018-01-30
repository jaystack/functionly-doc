# Classes
Available classes in Functionly

## Functionly core classes
- [Resource](#resource)
- [FunctionalService](#functionalservice)
- [Service](#service)
- [Api](#api)
- [Hook](#hook)
- [PreHook](#prehook)
- [PostHook](#posthook)

## AWS-related classes
- [DocumentClientApi](#documentclientapi)
- [DynamoTable](#dynamotable)
- [S3Api](#s3api)
- [S3Storage](#s3storage)
- [SNSApi](#snsapi)
- [SimpleNotificationService](#simplenotificationservice)
- [ApiGateway](#apigateway)


# Resource
This is the base class for every resource that Functionly can use. It manages the environment variables propagation to the [FunctionalServices](#functionalservice)
 
## FunctionalService
FunctionalServices are the entrypoints of the applications. These can subscribe to the event sources when annotated with [[rest|decorators#rest]] or [[eventSource|decorators#eventsource]] decorators. The services have to implement a `handle` method which contains a business logic. These can inject other resources ([Api](#api), [Service](#service) or an other [FunctionalService](#functionalservice)) or collect parameters ([[param|decorators#param]], [[stage|decorators#stage]], etc...)
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    async handle(@param name) {
        return `hello ${name}`
    }
}
```
> You have to export the `Home.createInvoker()` results from the `main.js` to publish the [FunctionalService](#functionalservice)

### Inject another FunctionalService
You can inject a FunctionalService from another FunctionalService. In AWS it's a cross-Lambda call, in a local environment it is a request to the other FunctionalService.
```js
@rest({ path: '/helloworld' })
class About extends FunctionalService {
    async handle(@param name, @inject(Home) home) {
        const homeResult = await home.invoke({ name })
        return { homeResult }
    }
}
```
!!! known issue: For the local environment: you have to decorate the injected [FunctionalService](#functionalservice) with the [[rest|decorators#rest]] decorator because it is called over the first exposed endpoint.

## Service
Services help to organize code you want reuse with [[inject|decorators#inject]]. Services are stateless but if you need states then use [Api](#api). Services have to implement a `handle` method. These services can only resolve invoke parameters with [[param|decorators#param]] decorator so they can't read for example from the request body.

### Definition
```js
@injectable()
class Greeter extends Service {
    async handle(@param name) {
        return `hello ${name}`
    }
}
```
### Usage
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    async handle(@param name, @inject(Greeter) greeter) {
        return await greeter({ name })
    }
}
```

## Api
APIs help to organize code you want reuse with [[inject|decorators#inject]]. Apis can store states but they can **disappear** any time because of the nature of serverless technology. In these classes you can inject other resources into the `constructor`. These have `init` functions which is an async method and can be invoked or waited for before the API is injected.
All other functions on these classes are simple JavaScript functions.

### Definition
```js
@injectable()
class NameValidator extends Service {
    async handle(@param name) {
        return name === 'John'
    }
}
@injectable()
class GreeterApi extends Api {
    constructor(@inject(NameValidator) private nameValidator) { }
    async init() { }

    async hello(name) {
        if (await this.nameValidator(name))
            return `hello ${name}`
        return `hello anonymous`
    }
    async bye(name) {
        return `bye ${name}`
    }
}
```
### Usage
```js
@rest({ path: '/helloworld' })
class Home extends FunctionalService {
    async handle(@param name, @inject(GreeterApi) greeter) {
        return await greeter.hello(name)
    }
}
```


## Hook
Hook is a base class of hooks. If used as a base class, it works like [PreHook](#prehook).

## PreHook
Hooks are middleware in Functionly, and you can chain them with the [[use|decorators#use]] decodator on [FunctionalServices](#functionalservice) or another [Hook](#hook). PreHooks are running `before` the decorated class. With the `@use(ResolveUser)` decoration the `ResolveUser` hook can be bound to the `Home` FunctionalService, and so the `@inject(ResolveUser) user` parameter of the `Home`'s handle method can receive the result.
```js
@injectable()
class ResolveUser extends PreHook {
    async handle(@param name, @param age) {
        return { name, age }
    }
}

@rest({ path: '/helloworld' })
@use(ResolveUser)
class Home extends FunctionalService {
    async handle(@param name, @inject(ResolveUser) user) {
        console.log(user)
    }
}
```


## PostHook
Hooks are middleware in Functionly, and you can chain them with the [[use|decorators#use]] decodator on [FunctionalServices](#functionalservice) or another [Hook](#hook). PostHooks are running `after` the decorated class. With the `@use(TransformResult)` decoration the `TransformResult` hook can be bound to the `Home` FunctionalService.
```js
class TransformResult extends PostHook {
    async handle(@result result) {
        return { result }
    }

    async catch(@error error) {
        return { error }
    }
}

@rest({ path: '/helloworld' })
@use(TransformResult)
class Home extends FunctionalService {
    async handle() {
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
> In a local environment it connects to `http://localhost:8000`. What you can change when you set the `DYNAMODB_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## DynamoTable
When you define a `DynamoTable` and it is related to any published [FunctionalService](#functionalservice) the deployment will create a DynamoDB table for the application. And you can [[inject|decorators#inject]] the `Users` table to use it like an [Api](#api). There are functions on it to work with records.
```js
@dynamoTable({ tableName: '%ClassName%-table'})
class Users extends DynamoTable {}
```


## S3Api
S3Api creates the `AWS.S3` connection to [S3Storage](#s3storage).
> In a local environment it connects to `http://localhost:4572`, which you can change when you set the `S3_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## S3Storage
When you define a `S3Storage` and it is related to any published [FunctionalService](#functionalservice) the deployment will create an S3 Bucket for the application. Then you can [[inject|decorators#inject]] the `Files` storage to use it like an [Api](#api). There are functions on it to work with files.
```js
@s3Storage({ bucketName: '%ClassName%-store'})
class Files extends S3Storage {}
```


## SNSApi
S3Api creates the `AWS.SNS` connection to [SimpleNotificationService](#simplenotificationservice).
> In a local environment it connects to `http://localhost:4100`, which you can change when you set the `SNS_LOCAL_ENDPOINT` environment variable. Also you can set the region with the `AWS_REGION`


## SimpleNotificationService
When you define a `SimpleNotificationService` and it's connected to any published [FunctionalService](#functionalservice), the deployment will create a Simple Notification Service topic for the application. And you can [[inject|decorators#inject]] the `Notifications` storage to use it like an [Api](#api). There is a function on it to publish messages.
```js
@sns({ topicName: '%ClassName%-topic'})
class Notifications extends SimpleNotificationService {}
```
> During every deployment, a new topic is created because the CloudFormation can't update customly named topics. The created topic name will contain an additional random number.


## ApiGateway
It is just a technical class for the [[eventSource|decorators#eventsource]] decorator. This allows you to subscribe to ApiGateway events like a DynamoTable record or S3 file changes. It does not work in a local environment like [[apiGateway|decorators#apigateway]].
