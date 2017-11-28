#Parameters
This section will introduce where and how can you use the param decorators.
* [param decorator](#param-decorator)
* param usage in
    * [Services]()
    * [FunctionalServices]()
    * [Hooks]()

# param decorator
The [param]() property decorator is for resolve the parameters in [Services](), [FuntionalServices]() and [Hooks](). Parameters can be resolved from the trigger context. Trigger context can be a simple function call or can be environment specific like a web request, DynamoDB record change, S3 file event and so on.

## param in Services
At service invocation the first parameter of the call is an object what contains de parameters to the call.
```js
class Home extends FunctionalService {
    public async handle(@inject(MyService) service) {
        // service invocation
        await service({ username: 'John', age: 42 })
    }
}
```
The object properties will be binded to the expected parameters by name. The `username` property in the call will be bound to the expected `username` parameter.
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
> In the [Service]() the [param]() property decorator can read property from the trigger context just from the object from the call. When the logic expect some value from the environment (eg: query, body param) the caller have to pass it.

## param in FunctionalServices
## param in Api
There is no possibility to use [param](#param-decorator) decorator in Api. In `constructor` inject your required services and expect the requred parameters in Api's functions like a simple parameter.
```js
class MyApi extends Api {
    public myFunction(p1: string, p2: string) {
        return p1 + p2
    }
}
```
## param in Hooks
Everything works what in [FunctionalServices]

### The [result]() and [error]() parameter decorators in PostHook
These can transform the FunctionalService result. The `handle` method can use the [result]() parameter decorator to get the [FunctionalService]()'s (or the previous [PostHook]()'s) result. The `catch` can handle and transform errors and use the [error]() decorator to get the error from the PreHook, FunctionalService or the last PostHook's error. The return value will be the new result. Throwed error will be the new error.
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

## param in custom classes
There is no possibility to use [param](#param-decorator) decorator in the custom classes.

# param decorator configuration
Some parameter resolution implementation can be configured by parameter config.
