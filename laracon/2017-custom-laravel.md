# Custom Laravel

## The soul of customization
### Wherefore Custom Laravel
* We've got the best defaults.
* Convention over configuration.
* When helpful convention Equal configuration.

### Customizing Laravel: the basics
* Built-to-be-modified base code (/app/*)
* The most flexible container you've ever seen, and Illuminate Contracts.
* Macros, facades, re-binding, and more.
* Environment variables and conditional config
* It's Just PHP.

### What you choose to customize matters
* Taking it too far is a rite of passage. -- Jettrey Way

### Macors
* One fo the hardest things to discover in any new app is: "where/how is this happening".

### Prioritize Discoverability
* Write your apps so anyone who has read the Laravel docs can jump into them easily.

### The scale of discoverability
* What is this black magic => Procedural code in your face.

### Discoverability caveat
* Just because something is less discoverable doesn't mean it's bad.
* Sometimes the best tool isn't the most discoverable.
* Sometimes it's us that needs to change, not the tool.

### Discoverability disasters
* Overriding default methods like `create()`.
* Modifying the `request` object.
* Route::controller.
* Custom facades that aren't facades.
* Anything makes you go "that's so clever" instead of "that's so simple".
* A Laravel request will make many instance, but we only use `Tag::create(request()->all())`.

## Customization tips & tricks
### How to customize Laravel and how to do it responsibly

### Preface: the container and service providers.
#### The container in 5 minutes
* registering things into it.
* resolving things out of it.
* things Laravel resolves using it.
* autowiring vs. explicit binding
* service providers
* aliases and Facades

##### Registering things into container
* Key value store: put this thing at this address.
* "Adress": a string, usually a fully-qualified class name or a shortcut string like "logger".
* "Thing": usually either a class name, a Closure, or an object

> `app()->bind('address', Thing::Class);`
> `app()->bind('logger', Logger::class);`

##### Resolving things out of the container
##### Things Laravel resolves using container
* Anything Laravel creates itself it creates using the container.
* Controller, middleware, etc.
* Why we can type hint dependencies in a controller.
