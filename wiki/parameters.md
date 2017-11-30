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
The [[param|decorators#param]] property decorator is for resolve parameter values in [[Services|classes#service]], [[FunctionalServices|classes#functionalservice]] and [[Hooks|classes#hook]]. Parameters can be resolved from the invocation context. This can be a simple function call or can be environment specific like a web request, DynamoDB record change, S3 file event and so on.

### Parameters
There are several ways to use `param` decorator
```js
class Home extends FunctionalService {
    public async handle(
        @param('name') p1, // 1
        @param name, // 2
        @param({ name: 'name' }) p3, // 3
    ) { }
}
```
These three usage are equivalent. The `1` and `2` just shortcuts but in `3` you can define additional properties to resolve handler. Read more in [Configriation](#configuration) section.
> When you `minify` the code you have to set the name of the parameter because the original parameter names usualy change.

> !!! Known issue: When use a parameter destructuring in any parameter, you also have to set the parameter name for other properties also.

## param in FunctionalService
[[FunctionalService|classes#functionalservice]] has `handle` method where the property decorators are usable. The [[param|decorators#param]] property decorator can read from the event. That event can come be [[rest|decorators#rest]], [[apiGateway|decorators#apigateway]] or different [[eventSources|decorators#eventsource]]. You can expect parameters what come from the event source.
* [rest and apiGateway](#rest-and-apigateway)
* [DynamoTable](#dynamotable)
* [S3Storage](#s3storage)
* [SimpleNotificationService](#simplenotificationservice)

### rest and apiGateway
    
These are HTTP like event sources. This type of source usualy have body, query, url and headers parameters. The [[param|decorators#param]] decorator reads value from these. Resolution order is:
1. body
2. query parameters
3. url parameters
4. headers
> You have possibility to change the resolution source. Read more in [Configuration](#configuration) 

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
Functional service what resolve the `Authorization` token from header and the `username` from query
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
The [[param|decorators#param]] resolves value from the first record of the event when the [[eventSource|decorators#eventsource]] is a DynamoTable.
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
Functional service what resolve the `dynamodb` data
```js
class MyService extends FunctionalService {
    public async handle(@param dynamodb) {
        // after the call
        console.log(dynamodb.ApproximateCreationDateTime) // => 1511966820
    }
}
```
> You have possibility to change the resolution source. Read more in [Configuration](#configuration) 
### S3Storage
The [[param|decorators#param]] resolves value from the first record of the event when the [[eventSource|decorators#eventsource]] is a S3Storage.
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
Functional service what resolve the `s3` data
```js
class MyService extends FunctionalService {
    public async handle(@param s3) {
        // after the call
        console.log(s3.bucket.name) // => "filestorage-bucket-dev"
    }
}
```
> You have possibility to change the resolution source. Read more in [Configuration](#configuration)
### SimpleNotificationService
The [[param|decorators#param]] resolves value from the first record of the event when the [[eventSource|decorators#eventsource]] is a SimpleNotificationService.
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
Functional service what resolve the `Sns` data
```js
class MyService extends FunctionalService {
    public async handle(@param Sns) {
        // after the call
        console.log(Sns.Message) // => "my message Joh"
    }
}
```
> You have possibility to change the resolution source. Read more in [Configuration](#configuration)

## param in Service
[[Service|classes#service]] has `handle` method where the property decorators are usable. The [[param|decorators#param]] property decorator can read values from the invocation's parameters object which is the first parameter of the call. When the logic expect some value from the environment (eg: query, body parameter values) the caller have to pass it.

Before call the service, you have to [[inject|decorators#inject]] it. The injection result is a function. It has a special signature: the first parameter of the call is an object what contains the parameters.
```js
class Home extends FunctionalService {
    public async handle(@inject(MyService) service) {
        // service invocation
        await service({ username: 'John', age: 42 })
    }
}
```
And in the service side, you can expect parameters. The object properties of the call will be binded to the expected parameters by name. The value of the `username` property in the parameter object will be bound to the first parameter which name is `username`.
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
Everything works what in [FunctionalService](#param-in-functionalservice), but the [[PostHook|classes#posthook]] can define `catch` handler to handle errors and the property decorators works there also.

### The [[result|decorators#result]] and [[error|decorators#error]] parameter decorators in PostHook
These are special value resolvers. The `handle` method can use the [[result|decorators#result]] parameter decorator to get the [[FunctionalService|classes#functionalservice]]'s (or the previous [[PostHook|classes#posthook]]'s) result. The `catch` can handle and transform errors and use the [[error|decorators#error]] decorator to get the error from the PreHook, FunctionalService or the last PostHook's error. The return value will be the new result. Throwed error will be the new error.
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
There is no possibility to use [param](#param-decorator) decorator in [[Api|classes#api]]. In `constructor` inject your required services and expect the requred parameters in Api's functions like a simple parameter.
```js
class MyApi extends Api {
    public myFunction(p1: string, p2: string) {
        return p1 + p2
    }
}
```

## param in custom classes
There is no possibility to use [param](#param-decorator) decorator in the custom classes.

# Configuration
The shortest way to use the [[param|decorators#param]] decorator is `@param name`. But when you want a different name for your variable, you can use `@param('name') myParameterName` format.
> When you `minify` the code you have to set the name of the parameter because the original parameter names usualy change.

But if you use the third `@param({ name: 'name' })` format, you can pass more property to param decorator.
## Properties
### name
> type: string

> optional

The name property in the configuration contains the property name of the bounded value. For example, the name of the header when you want to get the header value. If it not set then the name of the expected parameter. 

The name property can be refer to the deep property of the event. If the path not exists the value is `undefined`.

Extended [DynamoTable](#dynamotable) example
```js
class MyService extends FunctionalService {
    public async handle(@param('dynamodb.ApproximateCreationDateTime') time) {
        // after the call
        console.log(time) // => 1511966820
    }
}
```
> Deep reference can handle objects only.

### source
> type: string

> optional

> This settings can be environment specific so if you change this, it maybe not work in an other environment!

Each event source handler has an own strategy resolve parameters. But with the source property you can change that.
1. When the value is `null` then the resolution source will be an object what contains the original event parameters. Same as what the [serviceParam](#serviceparam) decorator returns.

    Extended [rest and apiGateway](#rest-and-apiGateway) example, get the username from the original event parameter of the event context. 
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

    Extended [rest and apiGateway](#rest-and-apiGateway) example, get the username from custom source. 
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
