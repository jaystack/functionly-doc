# Decorators
Available decorators in Functionly

# Class decorators
- [apiGateway](#apigateway)
- [aws](#aws)
- [description](#description)
- [dynamoTable](#dynamotable)
- [environment](#environment)
- [eventSource](#eventsource)
- [functionName](#functionname)
- [injectable](#injectable)
- [log](#log)
- [rest](#rest)
- [role](#role)
- [s3Storage](#s3storage)
- [sns](#sns)
- [tag](#tag)
- [use](#use)

# Parameter decorators
- [inject](#inject)
- [param](#param)
- [serviceParams](#serviceparams)
- [request](#request)
- [result](#result)
- [error](#error)
- [functionalServiceName](#functionalservicename)
- [provider](#provider)
- [stage](#stage)

# Decorators

## apiGateway
Define an API gateway endpoint for Lambda
> usable at [[FunctionalService|classes#functionalservice]]

> **AWS** only, use [rest](#rest) for cross environment

### Parameters
```js
@apiGateway({
    path: string,
    method?: string = 'get',
    cors?: boolean = true,
    authorization?: 'AWS_IAM' | 'NONE' | 'CUSTOM' | 'COGNITO_USER_POOLS' = 'AWS_IAM'
})
```

### Example
API gateway endpoint in `/home` path. Your function will be available at `https://xyz.execute-api.us-east-1.amazonaws.com/dev/home`. Path value can contain parameters and can be deep like `/home/view/{id}`
```js
@apiGateway({ path: '/home' })
class Home extends FunctionalService {}
```


## aws
You can change AWS Lambda basic settings
> usable at [[FunctionalService|classes#functionalservice]]

> **AWS** only

### Parameters
```js
@aws({
    type?: 'nodejs6.10' = 'node6.10',
    memorySize?: number, // AWS default
    timeout?: number // AWS default
})
```

### Example
```js
@aws({ timeout: 10 })
class Home extends FunctionalService {}
```


## description
Description of the Functional Service
> usable at [[FunctionalService|classes#functionalservice]]

### Parameters
```js
@description(value: string)
```

### Example
```js
@description('description of the Home service')
class Home extends FunctionalService {}
```


## dynamoTable
Set up a Dynamo table for the application. It allows you to store data with the app.
> usable at [[DynamoTable|classes#dynamotable]]

### Parameters
```js
@dynamoTable({
    tableName: string,
    environmentKey?: string,
    nativeConfig?: any
})
```
> The created table will be suffixed with the stage name. Read more [[here|Deployment#stage]].

> In the `tableName` and the `environmentKey` properties can contain the `%ClassName%` placeholder and it will be replaced with the current class's name.

### Default values
#### environmentKey: 
This environment variable will contains the current table name
```js
%ClassName%_TABLE_NAME
```
#### nativeConfig
More information about the native (CloudFormation) configuration options and its values are [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)
```js
{
    AttributeDefinitions: [
        {
            AttributeName: "id",
            AttributeType: "S"
        }
    ],
    KeySchema: [
        {
            AttributeName: "id",
            KeyType: "HASH"
        }
    ],
    ProvisionedThroughput: {
        ReadCapacityUnits: 2,
        WriteCapacityUnits: 2
    }
}
```

### Example
```js
@dynamoTable({ tableName: 'Users-table' })
class Users extends DynamoTable {}
```


## environment
You can define an environment variable and the value for the resource.
> usable at [[Resource|classes#resource]]

### Parameters
```js
@environment(key: string, value: string)
```

### Example
```js
@environment('MY_ENVIRONMENT', '1')
class Home extends FunctionalService {}
```


## eventSource
There are more possibilities to trigger the FunctionalService. You can handle web requests but the code can run when a new record is inserted into the Dynamo table, a file uploaded into S3 or a simple notification service is invoked.
> usable at [[FunctionalService|classes#functionalservice]]

### Example
[AWS example](https://github.com/jaystack/functionly-examples/tree/master/eventSource)


## functionName
You can set a different name for the FunctionalService then one for the name of the class
> usable at [[FunctionalService|classes#functionalservice]]

> In **AWS** the created lambda will be suffixed with the stage name. Read more [[here|Deployment#stage]].

### Parameters
```js
@functionName(name: string)
```

### Example
```js
@functionName('Dashboard')
class Home extends FunctionalService {}
```


## injectable
There is a simple dependency injection system in Functionly. When you want to [inject](#inject) the code, you have to mark it with the injectable decorator.

### Injection modes
There is two injection scope the `Transient` and the `Singleton`.
```ts
enum InjectionScope {
    Transient = 1,
    Singleton = 2
}
```

### Parameters
default scope is `InjectionScope.Transient`
```js
@injectable(scope: InjectionScope)
```

### Example
```js
@injectable(InjectionScope.Singleton)
class Home extends FunctionalService {}

@injectable()
class MyApi extends Api {}
```


## log
In local environment it writes each request to the console
> usable at [[FunctionalService|classes#functionalservice]]

> **local** only

### Example
```js
@log()
class Home extends FunctionalService {}
```


## rest
Define a REST endpoint for the FunctionalService. This decorator is a generic way to define a REST endpoint if you use it in **AWS** environment it is works like an [apiGateway](#apigateway).
> usable at [[FunctionalService|classes#functionalservice]]

### Parameters
```js
@rest({
    path: string,
    methods?: string[],
    cors?: boolean,
    anonymous?: boolean
})
```

### Example
REST endpoint under the `/home` path. Path value can contain parameters and can be deep like `/home/view/{id}`
```js
@rest({ path: '/home' })
class Home extends FunctionalService {}
```

## role
Set the AWS role name for the FunctionalService - without this, the name of role will be the name of the application, or if you set a specific AWS role resource ID, then that will be used. By default, Functionly will discover the used AWS resources and create a role for the application with the necessary policy.
> usable at [[FunctionalService|classes#functionalservice]]

> **AWS** only

### Parameters
```js
@role(value: string)
```

### Example
REST endpoint under the `/home` path. Path value can contain parameters and can be deep like `/home/view/{id}`
```js
@role('MyApplicationRole')
class Home extends FunctionalService {}

@role('arn:....')
class Home extends FunctionalService {}
```


## s3Storage
Set up an S3 file storage for the application. It allows you to store files with the app.
> usable at [[S3Storage|classes#s3storage]]

### Parameters
> bucket name has [naming rescricitons](http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html)
```js
@s3Storage({
    bucketName: string,
    environmentKey?: string,
    eventSourceConfiguration?: {
        Event?: any,
        Filter?: any
    }
})
```
> The created S3 file storage will be suffixed with the stage name. Read more [[here|Deployment#stage]].

> In the `bucketName` and the `environmentKey` properties can contain the `%ClassName%` placeholder and it will be replaced with the current class's name.

### Default values
#### environmentKey
```js
%ClassName%_S3_BUCKET
```
#### eventSourceConfiguration
This configuration is used when this decorator's target class is a parameter in [eventSource](#eventsource).
More info available [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-notificationconfig-lambdaconfig.html)
```js
{
    Event: "s3:ObjectCreated:*"
}
```

### Example
```js
@s3Storage({ tableName: 'file-storage' })
class FileStorage extends S3Storage {}
```

## sns
Set up an Simple Notification Service for the application. It allows you to send notifications with the app.
> usable at [[SimpleNotificationService|classes#simplenotificationservice]]

### Parameters
```js
@sns({
    topicName: string,
    environmentKey?: string
})
```
> The SNS Topic name will be suffixed with the stage name. Read more [[here|Deployment#stage]].

> The created SNS Topic name will contains an additional random number.

> In the `topicName` and the `environmentKey` properties can contain the `%ClassName%` placeholder and it will be replaced with the current class's name.

### Default values
#### environmentKey
```js
%ClassName%_SNS_TOPICNAME
```

### Example
```js
@sns({ topicName: 'my-topic'})
class Notifications extends SimpleNotificationService {}
```

## tag
You can define a tag and the value for the FunctionalService.
> usable at [[FunctionalService|classes#functionalservice]]

### Parameters
```js
@tag(key: string, value: string)
```

### Example
```js
@tag('mytag', 'value')
class Home extends FunctionalService {}
```


## use
There is a middleware system in Functionly. You can set [[PreHook|classes#prehook]] and [[PostHook|classes#posthook]] as middleware for the [[FunctionalService|classes#functionalservice]]. You can read more about middleware [[here|middleware]], or check out the unit tests [here](https://github.com/jaystack/functionly/blob/master/test/hook.tests.ts#L677)
> usable at [[FunctionalService|classes#functionalservice]]

> usable at [[Hook|classes#hook]]

### Parameters
```js
@use(...middleware)
```

### Example
The `Authentication` and the `RoleCheck` are [PreHook|classes#prehook]]s and the `ErrorHandler` is a [[PostHook|classes#posthook]]
```js
@use(Authentication)
@use(RoleCheck)
@use(ErrorHandler)
class Home extends FunctionalService {}
```
these are equvivalents
```js
@use(Authentication, RoleCheck, ErrorHandler)
class Home extends FunctionalService {}
```


## inject
There is a simple dependency injection system in Functionly. After you mark the class with `injectable` decorator, you can inject into the code. Read more about injection [[here|dependency-injection]]

### Parameters
```js
@inject(MyClass)
```

### Example
```js
class Home extends FunctionalService {
    async handle(@inject(Api) api) { }
}
```


## param
With `param` parameter decorator, you can resolve values from event sources. For example, from web requests you can get values from query parameters, from the request body or from the request header. Read more [[here|parameters]] about how it's working.

### Parameters
```js
@param(options?)
```

### Example
There are several ways to use `param` decorator
```js
class Home extends FunctionalService {
    async handle(
        @param('name') name,
        @param name,
        @param({ property: 'myValue' }) age,
    ) { }
}
```
The `@param name` is a shortcut here for `@param('name') name`.


## serviceParams
By using this, you can get the original parameters of the event source. In the **AWS** environment it has properties like `event`, `context` and `cb`. In **local** it has `req`, `res` and `next`.

### Example
```js
class Home extends FunctionalService {
    async handle(@serviceParams rawServiceParameters) { }
}
```


## request
With the `request` decorator, you can get request data from both local and AWS requests in the same object format. It returns `undefined` from event sources which are not web request-based.

### Interface
```js
{
    url: Url,
    method: string,
    body: Object | string,
    query: Object,
    params: Object,
    headers: Object
}
```

### Example
```js
class Home extends FunctionalService {
    async handle(@request requestData) { }
}
```


## result
The `result` decorator can resolve the previous middleware result. For example, if you use it within [[PostHook|classes#posthook]]middleware you can transform the FunctionalService response

### Example
```js
class ResponseTransform extends PostHook {
    async handle(@result result) { }
}
```


## error
It is only usable in [[PostHook|classes#posthook]]'s catch handler to handle errors in the FunctionalService

### Example
```js
class ResponseTransform extends PostHook {
    async catch(@error error) { }
}
```


## functionalServiceName
Get the [[FunctionalService|classes#functionalservice]]'s name dynamically with `functionalServiceName` decorator in [[FunctionalService|classes#functionalservice]] or in any type of [[Hook|classes#hook]]

### Example
```js
class Home extends FunctionalService {
    async handle(@functionalServiceName name) { }
}
```


## provider
Get the current environment name. It is `local` in a local environment or `aws` when it runs in AWS

### Example
```js
class Home extends FunctionalService {
    async handle(@provider providerKey) { }
}
```


## stage
Get the current stage name.

### Example
```js
class Home extends FunctionalService {
    async handle(@stage stage) { }
}
```