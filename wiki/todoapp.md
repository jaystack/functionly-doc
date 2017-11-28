# Todo app with functionly
This quick start guide will teach you how to create Todo app with [Functionly](https://www.npmjs.com/package/functionly).

In this tutorial you will:

* [Create a dynamo table](#create-a-dynamo-table)
* [Create functional services](#create-functional-services)
* [Read todos](#read-todos)
* [Create todo](#create-todo)
* *[Extend with Services](#extend-the-example-with-Services) - optional*
* [Run and Deploy with CLI](#run-and-deploy-with-cli)
* [AWS deployment](#aws-deployment)

## Create a dynamo table
We need a DynamoTable because we want to store todo items. It will be the `TodoTable`.
```js
import { DynamoTable, dynamoTable, injectable } from 'functionly'

@injectable()
@dynamoTable({ tableName: '%ClassName%_corpjs_functionly' })
export class TodoTable extends DynamoTable { }
```

## Create functional services
Define a base class for FunctionalService to set basic labda settings in AWS environment.
```js
import { FunctionalService, aws } from 'functionly'

@aws({ type: 'nodejs6.10', memorySize: 512, timeout: 3 })
export class TodoService extends FunctionalService { }
```

### Read todos
We need to create service to read a todos.
```js
export class GetAllTodos extends TodoService {
    public async handle() {}
}
```
Decorate it with the [rest]() decorator. We need a `path` and have to set the `cors` and the `anonymous` property to `true` because we want to call it without authentication and from other domain.
```js
@rest({ path: '/getAllTodos', cors: true, anonymous: true })
```
Define a [description]() to the `TodoService` it makes it easier to find in AWS Lambda list.
```js
@description('get all Todo service')
```
Now we have to create the business logic. We want to read the todos so inject the `TodoTable`. Get the items from it and return from service. If we not set the `methods` property that means its accept `GET` requests. (default: `methods: ['get']`)
```js
import { rest, description, inject } from 'functionly'

@rest({ path: '/getAllTodos', cors: true, anonymous: true })
@description('get all Todo service')
export class GetAllTodos extends TodoService {
    public async handle(@inject(TodoTable) db: TodoTable) {
        let items: any = await db.scan()
        return { ok: 1, items }
    }
}
```
We are almost done, but we have to export our service from the main file.
```js
export const getAllTodos = GetAllTodos.createInvoker()
```

### Create todo
We need a service to create todos, lets do this. Also define a [rest]() endpoint and a [description]().
```js
import { rest, description } from 'functionly'

@rest({ path: '/createTodo', methods: ['post'], anonymous: true, cors: true })
@description('create Todo service')
export class CreateTodo extends TodoService {
    public async handle() {}
}
```
We need some value to create a new todo: `name`, `description` and `status`. Expect these with the [param]() decorator and it will resolve them from the invocation context.
```js
import { rest, description, param } from 'functionly'

@rest({ path: '/createTodo', methods: ['post'], anonymous: true, cors: true })
@description('create Todo service')
export class CreateTodo extends TodoService {
    public async handle(@param name, @param description, @param staus) {}
}
```
The business logic: save a new todo item. [Inject]() the `TodoTable` and save a new todo with the `put` function. We need an id for the new todo i use [shortid](https://www.npmjs.com/package/shortid) for generate them.
```js
import { generate } from 'shortid'
import { rest, description, param } from 'functionly'

@rest({ path: '/createTodo', methods: ['post'], anonymous: true, cors: true })
@description('create Todo service')
export class CreateTodo extends TodoService {
    public async handle(@param name, @param description, @param status, @inject(TodoTable) db: TodoTable) {
        let item = {
            id: generate(),
            name,
            description,
            status
        }

        await db.put({ Item: item })

        return { ok: 1, item }
    }
}

export const createTodo = CreateTodo.createInvoker()
```

## Extend the example with Services
> **Optional**

Create two services: validate and persist todo. Then the CreateTodo only call these services.

### Validate todo
It will be an [injectable]() service and expect the three todo values then implement a validation logic in the service.
```js
import { injectable, param } from 'functionly'

@injectable()
export class ValidateTodo extends Service {
    public async handle( @param name, @param description, @param status) {
        const isValid = true
        return { isValid }
    }
}
```

### Persist todo
It will be an [injectable]() service and expect the three todo values and [inject]() a `TodoTable` then implement a persit logic in the service.
```js
import { injectable, param, inject } from 'functionly'

@injectable()
export class PersistTodo extends Service {
    public async handle( @param name, @param description, @param status, @inject(TodoTable) db: TodoTable) {
        let item = {
            id: generate(),
            name,
            description,
            status
        }
        await db.put({ Item: item })
        return item
    }
}
```

### Changed CreateTodo FunctionalService
[inject]() the two new services(`ValidateTodo`, `PersistTodo`) and change the business logic
```js
import { rest, description, param, inject } from 'functionly'

@rest({ path: '/createTodo', methods: ['post'], anonymous: true, cors: true })
@description('create Todo service')
export class CreateTodo extends TodoService {
    public async handle( 
        @param name, 
        @param description, 
        @param status, 
        @inject(ValidateTodo) validateTodo,
        @inject(PersistTodo) persistTodo
    ) {
        let validateResult = await validateTodo({ name, description, status })
        if (!validateResult.isValid) {
            throw new Error('Todo validation error')
        }
        let persistTodoResult = await persistTodo({ name, description, status })
        return { ok: 1, persistTodoResult }
    }
}
```

### The source code of this example is available [here](https://github.com/jaystack/functionly-examples/tree/master/todoDB)

# Install and Build
```sh
npm install
npm run build
```

# Run and Deploy with CLI
The CLI helps to deploy and run the application. 
1. CLI install
```sh
npm install functionly -g
```

## Local deployment
1. Create DynamoDB with docker
```sh
docker run -d --name dynamodb -p 8000:8000 peopleperhour/dynamodb
```
2. Deploy will create the tables in the DynamoDB
> Note: Create the [functionly.json](https://raw.githubusercontent.com/jaystack/functionly-examples/master/todoDB/functionly.json) in the project for short commands. And you don't have to pass all arguments.
```sh
functionly deploy local
```
## Run in local environment
In development time you can run the application in your local machine.
```sh
functionly local
```

## AWS deployment
> [Set up](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) the AWS Credential before deploy.

> Note: Create the [functionly.json](https://raw.githubusercontent.com/jaystack/functionly-examples/master/todoDB/functionly.json) in the project for short commands. And you don't have to pass all arguments. Because the `deployTarget` is configured to `aws` (the default value configured) then in the deploy command will use this as deploy target.

Functionly create the package and deploy the application to AWS. The package is a [CloudFormation](https://aws.amazon.com/cloudformation/) template, its contains AWS resources and the AWS can create or update the application's resources by the template.
```sh
functionly deploy
```

> Congratulation! Now you created and deployed your first `functionly` application!