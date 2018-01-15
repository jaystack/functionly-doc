*Abstract*
Nowadays, we have many options for implementing web services with scalable cloud providers. They provide similar interfaces to describe our business logic in an abstract way without scaling effort. Including the increasingly popular serverless architecture, these technologies allow for fast prototyping, even though the service components of any such provider have a steep learning curve for using it professionally. So we can declare the migration between two providers as the next technical effort.

Cloud technologies have already led us to abstract our implementation of business logic in the first order (logic abstraction), therefore why should we be bound to a particular provider? This is the second abstraction order; the provider abstraction.


[9:41] 
*Motivation*
For example serverless is a convenient, but limited framework. Your serverless code is not able to migrate to other cloud providers or even a dockerised express app. Your deployment flow is also very limited.

Functionly is a more innovative concept and framework. Its principles are:

- *Logic abstraction*: Hide the process and protocol handling. Describe your services in pure functions, and simply work with the efficient business data.
- *Provider abstraction*: Hide the infrastructure elements. Provide your side effects via an injection mechanism.
- *Unlimited control*: `Ensures access to the native and low-level implementations. Even one runtime/deploy lifecycle controlling or even native resource settings. But all of them must be defined strictly separated from your logic implementations.` _This doesn't make sense to me at all_
These principles guarantee your applications are independent and portable. (edited)


glenno [9:46 PM] 
*Logic abstraction*
Let's suppose we are developing a microservice. We all know a microservice defines a logic unit of the entire service and has a strictly defined scope within the system. In most cases, microservices are able to replace them gracefully without any downtime for the whole system.

Even though microservices' logic is defined definitely, you have to implement a lot of things on top of this logic for each service:


[9:49] 
Actually, the largest part of the service is not your logic. Including them as microservices means they cannot be micro anymore.

Serverless technologies allow you to forget items from this list, and ensures you can focus only on your service logic. Almost...


[9:50] 
*Provider abstraction*
Logic abstraction is great, but you still need to ensure the cloud environment with the following settings:

- continuous integration
- forming environment stages (dev, stage, prod, etc.)
- proper resource naming (db table names, queue names, etc.)
- auth and api gateway configurations
- role configurations
- security configurations
- cache configurations
- logging scopes
- scaling parameters
- etc


[9:53] 
Understanding each cloud environment could be a specialisation in itself meaning you need to bring in the skills for every provider you ever come across.

Our aim is to provide a solution via the three key principles of functionly.


[9:53] 
Functionly brings the functional approach to nodejs hosted conventional microservices - providing you an easy way to go serverless once you are ready to give up meddling with nodejs and docker.


[9:55] 
What happens exactly? Look at the code snippet above. We created a Login service. The most important aspect with functionly is we do not implement working code, we are simply meta-programming. Every service is a meta description and functionly is able to build working code in several environments. For example in an express application, it will look like to similar this: (edited)


[9:57] 
In AWS environments the handle method will be a lambda function, with POST /login routing.

This way, Login class never will be an instance. The static handle method is the only implementation (in our code) which is going to be used in the implementation of functionly.

The class is only necessary for describing meta informations with decorators. We decorate our services with metadata. This way a service will never be just a function. This is rather one Json object, with a pure logic implementation as a function, like the following:


[9:58] 
Functionly resolves the dependencies, ensures the proper roles for these dependencies, ensures the environment and resources, and wraps the implementations with the given environment. The result is environment specific code with resource descriptors already provided. This will deployed to the chosen provider.


[9:59] 
One of the primary advantages of functionly is the provider-independent code. You can deploy your meta-code to many providers. If you miss one of them, you are unrestricted in creating the connector as a plugin.


[10:01] 
*Easy to test*
The injection mechanism of functionly makes (easy to mock) dependencies for services, because service implementations are pure side-effect-less functions.

The components of your architecture form a dependency network. You can always slice a sub-network and test it separately, as a single service or even as a service-group.


[10:01] 
That's not perfect, but a solid improvement
