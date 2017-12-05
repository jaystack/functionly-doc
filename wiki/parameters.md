# Parameters
This section will introduce where and how can you use the param decorators.
* [param decorator](#param-decorator)
* param usage in
    * [FunctionalService](#param-in-functionalservice)
    * [Service](#param-in-service)
    * [Api](#param-in-api)
    * [Hook](#param-in-hook)
* [Configuration](#configuration)
* [serviceParam](#serviceparam)

# param decorator
The [[param|decorators#param]] property decorator is for resolving parameter values in [[Services|classes#service]], [[FunctionalServices|classes#functionalservice]] and [[Hooks|classes#hook]]. Parameters can be resolved using the invocation context. This can be a simple function call or can be an environment specific process like a web request, a DynamoDB record change, an S3 file event and so on.

### Parameters
There are several ways to use the `param` decorator
```js
class Home extends FunctionalService {
    public async handle(
        @param('name') p1, // 1
        @param name, // 2
        @param({ name: 'name' }) p3, // 3
    ) { }
}
```
These three usage samples are equivalent. `1` and `2` are just shortcuts but in `3` you can also define additional properties to resolve handlers. Read more in [Configriation](#configuration) section.
> When you `minify` the code you have to set the name of the parameter because the original parameter names usualy change.

> !!! Known issue: When you use parameter destructuring in any parameter, you also have to set the parameter name for other properties also.

## param in FunctionalService
[[FunctionalService|classes#functionalservice]] has `handle` methods where the property decorators can be used. The [[param|decorators#param]] property decorator can read from events. Events can come from [[rest|decorators#rest]], [[apiGateway|decorators#apigateway]] or different [[eventSources|decorators#eventsource]]. You can use parameters which are coming from the event source.
* [rest and apiGateway](#rest-and-apigateway)
* [DynamoTable](#dynamotable)
* [S3Storage](#s3storage)
* [SimpleNotificationService](#simplenotificationservice)

### rest and apiGateway
    
These are HTTP-like event sources. This type of source usually have body, query, url and headers parameters. The [[param|decorators#param]] decorator reads values from these. Resolution order is:
1. body
2. query parameters
3. url parameters
4. headers
> You have the option to change the resolution source. Read more at [Configuration](#configuration) 

#### Example
The Api Gateway event object
```js
{
    "resource": "/createItem",
    "path": "/createItem",
    "httpMethod": "GET",
    "headers": {
        "Authorization": "abc",
    },
    "queryStringParameters": {
        "username": "John"
    },
    "pathParameters": null,
    "stageVariables": null,
    "requestContext": {...},
    "body": null,
    "isBase64Encoded": false
}
```
Functional service that resolves the `Authorization` token from the header and the `username` from the query
```js
class MyService extends FunctionalService {
    public async handle(@param username, @param Authorization) {
        // after the call
        console.log(username)           // => 'John'
        console.log(Authorization)      // => 'abc'
    }
}
```

### DynamoTable
The [[param|decorators#param]] resolves values from the first record of the event when [[eventSource|decorators#eventsource]] is a DynamoTable.
```js
{
    "Records": [
        {
            "eventID": "6f0f2297c92c2651b813d3236c7c7d75",
            "eventName": "INSERT",
            "eventVersion": "1.1",
            "eventSource": "aws:dynamodb",
            "awsRegion": "eu-central-1",
            "dynamodb": {
                "ApproximateCreationDateTime": 1511966820,
                "Keys": {
                    "id": {
                        "S": "SJ2w_r2xG"
                    }
                },
                "NewImage": {
                    "name": {
                        "S": "John"
                    },
                    "id": {
                        "S": "SJ2w_r2xG"
                    }
                },
                "SequenceNumber": "300000000000067540029",
                "SizeBytes": 30,
                "StreamViewType": "NEW_AND_OLD_IMAGES"
            },
            "eventSourceARN": "arn:aws:dynamodb:..."
        }
    ]
}
```
Functional service that resolves the `dynamodb` data
```js
class MyService extends FunctionalService {
    public async handle(@param dynamodb) {
        // after the call
        console.log(dynamodb.ApproximateCreationDateTime) // => 1511966820
    }
}
```
> You have option to change the resolution source. Read more at [Configuration](#configuration) 
### S3Storage
The [[param|decorators#param]] resolves values from the first record of the event when the [[eventSource|decorators#eventsource]] is an S3Storage.
```js
{
    "Records": [
        {
            "eventVersion": "2.0",
            "eventSource": "aws:s3",
            "awsRegion": "xxx"
            "eventTime": "2017-11-29T14:47:00.248Z",
            "eventName": "ObjectCreated:Put",
            "userIdentity": {...},
            "requestParameters": {
                "sourceIPAddress": "x.x.x.x"
            },
            "responseElements": {...},
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "73d80f20-e894-46df-af29-63755051292a",
                "bucket": {
                    "name": "filestorage-bucket-dev",
                    "ownerIdentity": {
                        "principalId": "abc"
                    },
                    "arn": "arn:aws:s3:::filestorage-bucket-dev"
                },
                "object": {
                    "key": "testSJ2w_r2xG.txt",
                    "size": 12,
                    "eTag": "6a918be7ea744af8b4b0253eb93caa08",
                    "sequencer": "005A1EC864313AABD5"
                }
            }
        }
    ]
}
```
Functional service that resolves the `s3` data
```js
class MyService extends FunctionalService {
    public async handle(@param s3) {
        // after the call
        console.log(s3.bucket.name) // => "filestorage-bucket-dev"
    }
}
```
> You have the option to change the resolution source. Read more at [Configuration](#configuration)
### SimpleNotificationService
The [[param|decorators#param]] resolves values from the first record of the event when the [[eventSource|decorators#eventsource]] is a SimpleNotificationService.
```js
{
    "Records": [
        {
            "EventSource": "aws:sns",
            "EventVersion": "1.0",
            "EventSubscriptionArn": "arn:aws:sns:...",
            "Sns": {
                "Type": "Notification",
                "MessageId": "fe74f5c6-1ba0-504f-9dd1-a75ed10432eb",
                "TopicArn": "arn:aws:sns:...",
                "Subject": "my subject SJ2w_r2xG",
                "Message": "my message John",
                "Timestamp": "2017-11-29T14:47:00.353Z",
                "SignatureVersion": "1",
                "Signature": "abc",
                "SigningCertUrl": "https://sns.eu-central-1.amazonaws.com/xyz.pem",
                "UnsubscribeUrl": "https://sns.eu-central-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:...",
                "MessageAttributes": {}
            }
        }
    ]
}
```
Functional service that resolves the `Sns` data
```js
class MyService extends FunctionalService {
    public async handle(@param Sns) {
        // after the call
        console.log(Sns.Message) // => "my message Joh"
    }
}
```
> You have the option to change the resolution source. Read more at [Configuration](#configuration)

## param in Service
[[Service|classes#service]] has a `handle` method where you can use the property decorators. The [[param|decorators#param]] property decorator can read values from the invocation's parameter object which is the first parameter of the call. When the logic expects some values from the environment (eg: query, body parameter values) the caller has to pass them.

Before calling the service, you have to [[inject|decorators#inject]] it. The injection's result is a function. It has a special signature: the first parameter of the call is an object which contains the parameters.
```js
class Home extends FunctionalService {
    public async handle(@inject(MyService) service) {
        // service invocation
        await service({ username: 'John', age: 42 })
    }
}
```
You can expect parameters on the service side. The object properties of the call will be bound to the expected parameters by name. The value of the `username` property in the parameter object will be bound to the first parameter which name is `username`.
```js
@injectable()
class MyService extends Service {
    public async handle(@param username, @param age) {
        // after the call
        console.log(username)   // => 'John'
        console.log(age)        // => 42
    }
}
```

## param in Hook
Everything works here which does in [FunctionalService](#param-in-functionalservice), but the [[PostHook|classes#posthook]] can also define a `catch` handler to handle errors. Property decorators work there as well.

### The [[result|decorators#result]] and [[error|decorators#error]] parameter decorators in PostHook
These are special value resolvers. The `handle` method can use the [[result|decorators#result]] parameter decorator to get the result of [[FunctionalService|classes#functionalservice]] (or that of the previous [[PostHook|classes#posthook]]). `catch` can handle and transform errors and use the [[error|decorators#error]] decorator to get the error from the PreHook, from the FunctionalService or the last PostHook's error. The return value will be the new result. The error that is thrown will be the new error.
```js
class ResultTransform extends PostHook {
    public async handle(@result result) {
        console.log(result)
    }
    public async catch(@error error) {
        console.log(error)
    }
}
```

## param in Api
It's not possible to use the [param](#param-decorator) decorator in [[Api|classes#api]]. Inject your required services in `constructor` and expect the requred parameters in API's functions like a simple parameter.
```js
class MyApi extends Api {
    public myFunction(p1: string, p2: string) {
        return p1 + p2
    }
}
```

## param in custom classes
It's not possible to use [param](#param-decorator) decorator in custom classes.

# Configuration
The easiest way to use the [[param|decorators#param]] decorator is `@param name`. When you want a different name for your variable, you can use the `@param('name') myParameterName` format.
> When you `minify` the code, you have to set the name of the parameter because the original parameter names usually change.

If you use the third `@param({ name: 'name' })` format, you can pass more properties to the param decorator.
## Properties
### name
> type: string

> optional

The name property in the configuration contains the property name of the bound value. For example, the name of the header when you want to get the header value. If not set, it will be the name of the expected parameter. 

The name property can refer to the deep property of the event. If the path does not exist, the value is `undefined`.

Extended [DynamoTable](#dynamotable) example
```js
class MyService extends FunctionalService {
    public async handle(@param('dynamodb.ApproximateCreationDateTime') time) {
        // after the call
        console.log(time) // => 1511966820
    }
}
```
> Deep reference can only handle objects.

### source
> type: string

> optional

> These settings can be environment specific so if you change this, it might not work in another environment!

Each event source handler has its own strategy to resolve parameters, but with the source property you can change that.
1. When the value is `null` then the resolution source will be an object which contains the original event parameters. The same as what the [serviceParam](#serviceparam) decorator returns.

    Extended [rest and apiGateway](#rest-and-apiGateway) example: get the username from the original event parameter of the event context. 
    ```js
    class MyService extends FunctionalService {
        public async handle(
            @param({ source: null, name: 'event.event.queryStringParameters.username' }) username, 
            @param Authorization
        ) {
            // after the call
            console.log(username)           // => 'John'
            console.log(Authorization)      // => 'abc'
        }
    }
    ```
2. You can set a custom path of the resolution source

    Extended [rest and apiGateway](#rest-and-apiGateway) example: get the username from a custom source. 
    ```js
    class MyService extends FunctionalService {
        public async handle(
            @param({ source: 'event.event.queryStringParameters' }) username, 
            @param Authorization
        ) {
            // after the call
            console.log(username)           // => 'John'
            console.log(Authorization)      // => 'abc'
        }
    }
    ```

# serviceParam
With this parameter decorator, you can get the original parameters of the event source. Read more [[here|decorators#serviceparam]]
