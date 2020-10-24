This is my loopback 4 notes

## Application 

- Application class in the central of setting up all the module's components, controllers,
  servers and binding.
- Application extends 'Context' and provides the controls for starting and stopping and
  the associated servers
-  Loopback team encourage creating subclass that extends 'Application' for better
  organizing config and setup
- Application can be configured with constructors, bindings or a combination of both

	*** BINDING CONFIGURATIONS ***

	- Binding is recommended for setting up an application
	- 'Context' provide functions like ['.component', '.server', '.controller'] 

	- '.component' allows binding of component constructor within Application
	  instance's context
	- '.controller' allow binding of controller to the Application Context
	- '.servers' binding of servers, allow bulk binding :- use Array app.server([MyServer,
	  RestServer])


## Component  

- Component allows the extensibility of a loopback application     
- Component can have 
	- controller - array of controller classes
	- provider - map of providers to be bound to the application context s
	- classes - map of typescript classes to be bound to the application contexts
	- server - map of name/class pairs of serverss
	- lifeCycleObserver - map of name/class pairs of serverss
	- Services - Array of service classes or providers
	- binding - array of bindings to be added to the application contexts

	* All these properties contribute to the application to add additional feature and capabilities

## Context  

- An abstraction of all state and dependencies in our application
- Context is used in loopback to manage everything
- global registry for everything in an app (config, state, dependencies, classes, etc)
- IOC container used to inject dependencies into your code
- Have access to updated/real-time application and request state at all times
- items in  context are indexed via a key and bound to a BoundValue. A Bindingkey is simply a string
  value and is used to look up whatever you store along with the key. For example
- 'getSync' used to fetch the boundValue from the bindkey

Application-level context (global) 
- stores all initial and modified app states throughout the entire life of the app :- while process
  is alive
- Configured when application is created

Server-level context
- child of application-level context
- holds config specific to a particular server instance
- server level context will have an 'Application-level context' as its parent
	- which means all the bindings in the application level is valid in the server level context
	- Unless these bindings are being replaces in the server instance 

Request-level context
- Dynamically created for each incoming request
- extend the application level context to give you access to 'application-level dependencies during
  req/res lifecycle
- garbage are collected once the response is set for memory management
- 'this.ctx' is available in a sequence
```ts
import { DefaultSequence, RestBindings, RequestContext } from '@loopback/rest'

class MySequence extends DefaultSequence {
	async handle(context: RequestContext) {
		const req = await this.ctx.get(RestBindings.Http.REQUEST)
		const res = await this.ctx.get(RestBindings.Http.RESPONSE)
		this.send(res, `Hello ${req.query.name}`)
	}
}
```

Storing and Retrieving items from a context
- Items in context are indexed via key value pairs

```ts
// app level
const app = new Application();
app.bind('hello').to('world'); // BindingKey='hello', BoundValue='world'
console.log(app.getSync<string>('hello')); // => 'world'
```






