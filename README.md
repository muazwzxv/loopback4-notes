This is my loopback 4 notes

# Application 

- Application class in the central of setting up all the module's components, controllers,
  servers and binding.
- Application extends 'Context' and provides the controls for starting and stopping and
  the associated servers
-  Loopback team encourage creating subclass that extends 'Application' for better
  organizing config and setup
- Application can be configured with constructors, bindings or a combination of both

###  BINDING CONFIGURATIONS 
	- Binding is recommended for setting up an application
	- 'Context' provide functions like ['.component', '.server', '.controller'] 
	- '.component' allows binding of component constructor within Application
	  instance's context
	- '.controller' allow binding of controller to the Application Context
	- '.servers' binding of servers, allow bulk binding :- use Array app.server([MyServer, RestServer])

# Component  

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


# Context  

- An abstraction of all state and dependencies in our application
- Context is used in loopback to manage everything
- global registry for everything in an app (config, state, dependencies, classes, etc)
- IOC container used to inject dependencies into your code
- Have access to updated/real-time application and request state at all times
- items in  context are indexed via a key and bound to a BoundValue. A Bindingkey is simply a string
  value and is used to look up whatever you store along with the key. For example
- 'getSync' used to fetch the boundValue from the bindkey

###  Application-level context (global) 
- stores all initial and modified app states throughout the entire life of the app :- while process
  is alive
- Configured when application is created

###  Server-level context
- child of application-level context
- holds config specific to a particular server instance
- server level context will have an 'Application-level context' as its parent
	- which means all the bindings in the application level is valid in the server level context
	- Unless these bindings are being replaces in the server instance 

### Request-level context
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

### Storing and Retrieving items from a context
- Items in context are indexed via key value pairs
- We bind the 'world to the key 'hello' :- use getSync() to fetch the value
```ts
// app level
const app = new Application();
app.bind('hello').to('world'); // BindingKey='hello', BoundValue='world'
console.log(app.getSync<string>('hello')); // => 'world'
```

### Dependency Injection
- Configs are added to the context during app instantiation by dev
- When configs are registered, context provides way to use dependencies during runtime
```ts
import {inject, Application } from '@loopback/core'
const app = new Application()
app.bind('defaultName').to('John')

export class HelloController {
	constructor(@inject('defaultName') private name: string) {}

	greet(name?: string) {
		return `Hello ${name || this.name}`
	}
}
```

### Context metadata and sugar decorators
 - @get :- tells loopback to trigger a certain function at HTTP get req
 - @param :- requesting for params arguments

###    Context events
			 - Instance of CONTEXT can emit the following events
			 - bind: Emitted when new binding addes to the context
			 - Unbind: Emitted when existing binding is removed from the context
			 - error: Emitted when an observer throws an error during notification process

### Context Observer
- **ContextObserverFn** :- Type
- **ContextObserver** :- Interface
- **ContextEventObserver** :- Async and invoked by notification queue after event is emitted (after
	emit returns)
- **ContextEventListenet** :- Async and invoked when the even is emitted (Before emit returns)
	- **Context API**
		- subscribe(observer: ContextEventObserver) :- Add context event observer to the context chain,
			ancestor included
		- unsubscribe(observer: ContextEventObserver) :- Remove context event observer from the context
			chain
		- close() :- Close the context and release reference to other object in the context chain
		- child context registers even listeners with its parents context, close() method must be called
			to avoid memory if child context is to be recycled
- To react on context event asyncly, need to implement **ContextObserver** interface or provide
	**ContextObserverFn** and register it with the context

```ts
const app = new Context('app')
server = new Context(app, 'server')

const observer: ContextObserver = {
	// Only interested in bindings tagged with `foo`
	filter: binding => binding.tagMap.foo != null,

	observe(event: ContextEventType, binding: Readonly<Binding<unknown>>) {
		if (event === 'bind') {
			console.log('bind: %s', binding.key)
			//... Perform async operations
		} else if(event === 'unbind') {
			console.log('unbind %s', binding.key)	
		}
	},
}

server.subscribe(observer)
server.bind('foo-server').to('foo-value').tag('foo')
app.bind('foo-app').to('foo-value').tag('foo')

/* Output
bind: foo-server
bind: foo-app
*/
```






