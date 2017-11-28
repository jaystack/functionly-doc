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
> usable at [FunctionalService]()

> **AWS** only, use [rest](#rest) for cross environment

### Parameters
```js
@apiGateway({
    path: string,
    method?: string,
    cors?: boolean,
    authorization?: 'AWS_IAM' | 'NONE' | 'CUSTOM' | 'COGNITO_USER_POOLS'
})
```

### Example
API gateway endpoint in `/home` path. Your function will be available at `https://xyz.execute-api.eu-central-1.amazonaws.com/dev/home`. Path value can contain parameters and can be deep like `/home/view/{id}`
```js
@apiGateway({ path: '/home' })
class Home extends FunctionalService {}
```


## aws
You can change AWS Lambda basic settings
> usable at [FunctionalService]()

> **AWS** only

### Parameters
```js
@aws({
    type?: 'nodejs6.10',
    memorySize?: number,
    timeout?: number
})
```

### Example
```js
@aws({ timeout: 10 })
class Home extends FunctionalService {}
```


## description
Description of the Functional Service

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
> usable at [DynamoTable]()

### Parameters
```js
@dynamoTable({
    tableName: string,
    environmentKey?: string,
    nativeConfig?: any
})
```

### Default values
#### environmentKey: 
!!! known issue: environmentKey must end with `_TABLE_NAME`
```js
%ClassName%_TABLE_NAME
```
#### nativeConfig
More information about the native configuration options and its values are [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)
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
> usable at [Resource]()

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
> usable at [FunctionalService]()

### Example
[AWS example](https://github.com/jaystack/functionly-examples/tree/master/eventSource)


## functionName
You can set a different name for the FunctionalService then one for the name of the class
> usable at [FunctionalService]()

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
> usable at [FunctionalService]()

> **local** only

### Example
```js
@log()
class Home extends FunctionalService {}
```


## rest
Define a REST endpoint for the FunctionalService
> usable at [FunctionalService]()

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
> usable at [FunctionalService]()

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
Set up an S3 environment for the application. It allows you to store files with the app.
> usable at [S3Storage]()

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

### Default values
#### environmentKey
!!! known issue: environmentKey must end with `_S3_BUCKET`
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


## tag
You can define a tag and the value for the FunctionalService.
> usable at [FunctionalService]()

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
There is a middleware system in Functionly. You can set [PreHooks]() and [PostHooks]() as middleware for the [FunctionalService](). You can read more about middlewares [here](), or check out the unit tests [here](https://github.com/jaystack/functionly/blob/master/test/hook.tests.ts#L677)
> usable at [FunctionalService]()

> usable at [Hook]()

### Parameters
```js
@use(...middlewares)
```

### Example
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
There is a simple dependency injection system in Functionly. After you mark the class with `injectable` decorator, you can inject into the code. Read more about injection [here]()

### Parameters
```js
@inject(MyClass)
```

### Example
```js
class Home extends FunctionalService {
    public async handle(@inject(Api) api) { }
}
```


## param
With `param` parameter decorator, you can resolve values from event sources. For example, from web requests you can get values from query parameters, from the request body or from the request header. Read more [here]() about how it's working.

### Parameters
```js
@param(options?)
```

### Example
There are two ways to use `param` decorator
```js
class Home extends FunctionalService {
    public async handle(
        @param name,
        @param({ property: 'myValue' }) age,
    ) { }
}
```


## serviceParams
By using this, you can get the original event source. In the **AWS** environment it has properties like `event`, `context` and `cb`. In **local** it has `req`, `res` and `next`.

### Example
```js
class Home extends FunctionalService {
    public async handle(@serviceParams rawServiceParameters) { }
}
```


## request
With the `request` decorator, you can get request data from both local and AWS requests in the same object format. It returns `undefined` from event sources which are not web request-based.

### Interface
```js
{
    url: Url,
    method: string,
    body: Object,
    query: Object,
    params: Object,
    headers: Object
}
```

### Example
```js
class Home extends FunctionalService {
    public async handle(@request requestData) { }
}
```


## result
The `result` decorator can resolve the previous middleware result. For example, if you use it within [PostHook]() middleware you can transform the FunctionalService response

### Example
```js
class ResponseTransform extends PostHook {
    public async handle(@result result) { }
}
```


## error
It is only usable in [PostHook]()'s catch handler to handle errors in the FunctionalService

### Example
```js
class ResponseTransform extends PostHook {
    public async catch(@error error) { }
}
```


## functionalServiceName
Get the [FunctionalService]()'s name dynamically with `functionalServiceName` decorator in [FunctionalService]() or in any type of [Hooks]()

### Example
```js
class Home extends FunctionalService {
    public async handle(@functionalServiceName name) { }
}
```


## provider
Get the current environment name. It is `local` in a local environment or `aws` when it runs in AWS

### Example
```js
class Home extends FunctionalService {
    public async handle(@provider providerKey) { }
}
```


## stage
Get the current stage name.

### Example
```js
class Home extends FunctionalService {
    public async handle(@stage stage) { }
}
```