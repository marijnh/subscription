# Subscription

A subscription is an object that emits events. Rather than overloading
objects with other roles to emit events, and having a single object
emit several types, as in `.addEventListener` style events, a
subscription is a dedicated object representing a single event source.

    // Instead of adding listeners by string type
    object.addEventListener("message", m => { ... })
    
    // You create a subscription object when you construct the base
    // object...
    class MyObject {
      constructor() {
        this.message = new Subscription
        ...
      }
    }
    
    // ...and add handlers directly to that:
    myObject.message.add(m => { ... })

The interface of a subscription is roughly based on
[`js-signals`](https://millermedeiros.github.io/js-signals/), but
whereas `js-signals` have a fixed dispatch algorithm, subscriptions
come in several forms.

The perceived benefits of subscriptions are that they are easier to
check statically (instead of a single method dispatching on a bunch of
strings, you have actual objects representing your event types, each
of which expects a single type of handler function), and lead to more
explicit code.

## Distribution

This module is licensed under an MIT license. The recommended way to
install it is with [NPM](https://www.npmjs.com/package/subscription).

## API

### class `Subscription`

A subscription is an object that you can add subscribers (functions)
to, which will be called every time the subscription is dispatched.

**`add`**`(f: Function, priority: ?number)`

Add a function of the appropriate type for this subscription to be
called whenever the subscription is dispatched. When `priority` is
provided (default is zero), it determines when the function is called
relative to other handlers—those with a high priority come first.

**`addOnce`**`(f: Function, priority: ?number)`

Add a function to be called once, the next time this subscription is
dispatched.

**`remove`**`(f: Function)`

Remove the given subscriber from the subscription.

**`hasHandler`**`() → bool`

Returns true if there are any functions registered with this
subscription.

**`dispatch`**`(...args: [any])`

Call all handlers for this subscription with the given arguments.

### class `PipelineSubscription` extends `Subscription`

A type of subscription that passes the value it is dispatched with
through all its subscribers, feeding the return value from each
subcriber to the next one, and finally returning the result.

**`dispatch`**`(value: any) → any`

Run all handlers on the given value, returning the result.

### class `StoppableSubscription` extends `Subscription`

A stoppable subscription is a subscription that stops calling further
subscribers as soon as a subscriber returns a truthy value.

**`dispatch`**`(...args: [any]) → any`

Call handlers with the given arguments. When one of them returns a
truthy value, immediately return that value.

### class `DOMSubscription` extends `Subscription`

A DOM subscription can be used to register intermediate handlers for
DOM events. It will call subscribers until one of them returns a
truthy value or calls `preventDefault` on the DOM event.

**`dispatch`**`(event: Event) → bool`

Run handlers on the given DOM event until one of them returns a truty
value or prevents the event's default behavior. Returns true if the
event was handled.
