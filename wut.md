## Abstract

Nowadays we have many abilities to **implement our web services within scalable cloud providers**. They provide similar interfaces to describe our business logic in an **abstract way** without scaling efforts. Including the more and more popular serverless technologies, these technologies allow the **fast prototyping**, even though that the service set of any such provider has a steep learning curve for using it professionally and deeply exploited. So we can declare the migration between two provider the next technical effort.

Cloud technologies once already led us to abstract our implementation of the business logic in first order (**logic abstraction**), then why should we bind us to a particular provider and adhere to that? This is the second abstraction order: the **provider abstraction**.

## Motivation

For example [serverless](https://www.npmjs.com/package/serverless) is a convenient, but also a very limited framework. Your serverless code is not able to migrate to an other cloud provider or even a dockerized express app. Your deployment flow is also very limited.

[Functionly](https://www.npmjs.com/package/functionly) is a more innovative concept and framework. Its principles are:

1) **Logic abstraction**: Hide the process and protocol handling. Describe your services in pure functions, and just work with the efficient business data.
2) **Provider abstraction**: Hide the infrastructural elements. Provide your side effects via injection mechanism.
3) **Unlimited control**: Ensure the access to the native and low-level implementations. Even one runtime/deploy lifecycle controlling or even native resource settings.

These principles guarantee your independency and portability.

## Logic abstraction



## Provider abstraction

## [Functionly](https://www.npmjs.com/package/functionly)

The purpose is describing the **pure business logic as a Service** without handling the protocol or any outher technical overhead.

```js
@post('/login')
class Login extends Service {
  async handle(@param username, @param password, @inject(UserTable) users) {
    const user = await users.find({ username, password: md5(password) })
    if (!user) throw new Exception('Invalid username or password')
    return user
  }
}
```