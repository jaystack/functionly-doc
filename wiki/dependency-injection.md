# Dependency Injection (DI)
This section will introduce how the Dependency Injection works and how can you use it in [Functionly](https://www.npmjs.com/package/functionly)

* [Decorators](#decorators)
* [Lifetime Management](#lifetime-management)
* [Usage in functionly](#usage-in-functionly)
    * [Service resources](service-resources)
    * [Api](#api)
    * [Service](#services)
    * [FunctionalService](#FunctionalService)
    * [Hook](#prehook)
    * [Custom class](#custom-class)
* [IoC](#ioc)

## Decorators
### Injectable
The [injectable]() decorator is used to mark class what we want [inject](#inject) into [Api](#api), [Service](#service), [FunctionalService](#functionalservice)

What can be injectable:
* [Api](#api)
* [Service](#service)
* [FunctionalService](#functionalservice)
* [Hook](#hook)
* any [class](#custom-class)
 
### Inject
The [inject]() parameter decorator is for inject classes what is marked with [injectable](#injectable)

Where can i use inject
* [Api](#api)
* [Service](#service)
* [FunctionalService](#functionalservice)
* [Hook](#hook)

## Lifetime Management
Injection scopes for specify the lifetime of the injected instances. When the [injectable](#injectable) decorator is used there is a posibility to set the injection scope.
> Default injection scope is `InjectionScope.Transient`
```ts
enum InjectionScope {
    Transient = 1,
    Singleton = 2
}
```

### Transient
Every time a new instance of the injected type will be created when the service is requested from the container.
```js
@injectable()
```
or
```js
@injectable(InjectionScope.Transient)
```

### Singleton
Only one instance of the injected type will be create at first time of injection and will use all time when it is requested from the container
```js
@injectable(InjectionScope.Singleton)
```
# Usage in functionly
## Service resources
> Service resources (like [DynamoTable](), [S3Storage](), etc...) are just an [Apis](#api)
## Api
[Apis]() are the collection of functions what are contains business logic of the application.
### Inject usage in Api
Only the `constructor` function can handle functionly parameter decorators ([inject](), etc..). Parameter value will be an initialized instance of an injected type.
```js
@injectable()
class MyApi extends Api {
    private connection

    public constrictor(
        @inject(AnOtherApi) private api: AnOtherApi,
        @inject(ConnectorService) private connectorService
    ) { }

    private async init(){
        this.connection = this.connectorService({ serviceParam1: true })
    }

    /* custom function */
    public async hasConnection(){
        return this.connection ? true : false
    }
}
```
### Use injected Api
Lifecycle of the `MyApi` in functionly when it is injected.
1. [IoC](#ioc) resolve an instane of the Api -> `constructor` called
2. `init` method of the instance run if the instance created. It is an async function
3. Injection occurs -> run the code what is injected the Api
```js
class MyService extends Service {
    public async handle(@inject(MyApi) myApi: MyApi) {
        /* myApi already initialized */
        myApi.hasConnection() // -> true
    }
}
```

## Service
[Services]() are special classes those have a `handle` method what is contains the business logic.
### Inject usage in Services
Only `handle` function can handle parameter decorators like ([param](), [inject](), etc..). Parameter value will be an initialized instance of an injected type.
```js
class MyService extends Service {
    public async handle(@inject(MyApi) myApi: MyApi) {
        /* myApi already initialized */
        myApi.hasConnection() // -> true
    }
}
```
### Use injected Services
Injected value will be an async function what is call the service invoke method. That function has a differenst signature then the `handle` method, it recives az object where object keys are the parameter's names
```js
@injectable()
class MyService extends Service {
    public async handle(@param p1: string, @param p2: string) {
        return p1 + p2
    }
}

class Home extends FunctionalService {
    public async handle(@inject(MyService) service) {
        /* service is a funtion */
        const result = await service({ p1: 'value1', p2: 'value2' })
        console.log(result) // -> 'value1value2'
    }
}
```
## FunctionalService
[FunctionalServices]() are special classes those have a `handle` method what is contains the business logic.
### Inject usage in FunctionalServices
Only `handle` function can handle parameter decorators like ([param](), [inject](), etc..). Parameter value will be an initialized instance of an injected type.
```js
class Home extends FunctionalService {
    public async handle(@inject(MyApi) myApi: MyApi) {
        /* myApi already initialized */
        myApi.hasConnection() // -> true
    }
}
```
### Use injected FunctionalServices
Injected value will be an instance of the injected type. That contains a `handle` method, it recives az object where object keys are the parameter's names.
In different environment it has different implementations.
* in **AWS** it is a lambda call to the injected lambda function
* in **local** it is a new request to the injected service 
> Note: in **local** environment it is works only if it has a [rest]() endpoint
```js
@injectable()
class Validate extends FunctionalService {
    public async handle(@param p1: string, @param p2: string) {
        return p1 + p2 !== p2 + p1
    }
}

class Home extends FunctionalService {
    public async handle(@inject(Validate) validate: Validate) {
        /* service is a funtion */
        const result = await validate.invoke({ p1: 'value1', p2: 'value2' })
        console.log(result) // -> true
    }
}
```
## Hook
Functionly supports [PreHooks]() and [PostHooks]() these also can be injectable and can inject other types.
### Inject usage in FunctionalServices
Both type of Hooks have `handle` method and can inject classes.
```js
class Auth extends PreHook {
    public async handle(@inject(MyApi) myApi: MyApi) {
        /* myApi already initialized */
        myApi.hasConnection() // -> true
    }
}
```
[PostHooks]() has a `catch` method what also can use inject parameter decorators.
```js
class MyPostHook extends PostHook {
    public async catch(@inject(MyApi) myApi: MyApi) {
        /* myApi already initialized */
        myApi.hasConnection() // -> true
    }
}
```

### Use injected custom class
PreHook injection has a special resolution, it is not returns the instance of the injected type but it is the last result of the type.
```js
@injectable()
class HelloWorld extends PreHook {
    public async handle(@param name) {
        return `hello ${name}`
    }
}

@use(HelloWorld)
class Home extends FunctionalService {
    public async handle(@inject(HelloWorld) helloWorld){
        console.log(helloWorld) // => 'hello functionly' if the name parameter is 'functionly'
        return helloWorld
    }
}
```

## Custom class
Any javascript class. [IoC](#ioc) creates an instance of the injected type. 
> Parameter decorators are not supported here.
### Use injected custom class
Lifecycle of the class in functionly when it is injected.
1. [IoC](#ioc) resolve an instance -> `constructor` invocation
3. Injection occurs -> run the code what is injected the Api
```js
class MyClass {
    public hello(name) {
        return `hello ${name}`
    }
}

class MyService extends Service {
    public async handle(@inject(MyClass) myClass: MyClass) {
        myClass.hello('functionly') // -> 'hello functionly'
    }
}
```

# IoC
There is an IoC container in function which helps to create new instances from types.
```js
import { container } from 'functionly'
```
## Functions
1.  resolve
    > signature: Class => instance

    Creates a new instance (depends on injection scope) of the given class
    ```js
    const user = container.resolve(User)
    ```
2. containsInstance
    > signature: Class => boolean

    Returns `true` if the container has a cached instance of the class what the resolve will return. Otherwise return `false`. If the injection scope of the parameter class is [Transient](#transient) then it is returns `false` every time. But when [Singleton](#singleton) it can returns `true` when the class already resolved at least one time.
    ```js
    const hasUser = container.containsInstance(User)
    ```
3. registerInstance
    > signature: (Class, instance) => void

    Register the instance for the class and the `resolve` will return it depends on the injection scope.
    ```js
    const user = new User()
    container.registerInstance(User, user)
    ```
4. registerType
    > signature: (from: Class, to: Class) => void

    Remap the class type in the container and then the `resolve` will return the instance of the new Class when the `resolve` calls with the original Class
    ```js
    container.registerType(User, WebUser)
    const user = container.resolve(User)
    console.log(user instanceof WebUser) // => true
    ```
5. resolveType
    > signature: (Class) => Class

    Returns the registered type for the given class. If it is not remapped them return it.
    ```js
    container.resolveType(User) // => User

    container.registerType(User, WebUser)
    container.resolveType(User) // => WebUser
    ```