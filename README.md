Mediator.js
===========

Version 0.9.5

Usage
-----

### Using in the Browser

```html
<script src="/js/Mediator.min.js"></script>

<script>
  var Mediator = require("mediator-js").Mediator,
      mediator = new Mediator();

  mediator.subscribe("wat", function(){ console.log(arguments); });
  mediator.publish("wat", 7, "hi", { one: 1 });
</script>
```

### API

You can register events with the mediator two ways using channels. You can add
a predicate to perform more complex matching.  Instantiate a new mediator, and
then you can being subscribing, removing, and publishing.


Subscription signature:
    var mediator = new Mediator();

    mediator.subscribe(channel, callback, <options>, <context>);
    mediator.publish(channel, <data, data, ... >)
    mediator.remove(channel, <identifier>)

Additionally, `on` and `bind` are aliased to `subscribe`, and `trigger` and
`emit` are bound to `publish`. `off` is an alias for `remove`. You can use
`once` to subscribe to an event that should only be fired once.

Subscriber signature:

    function(<data, data ...>, channel);

The channel is always returned as the last argument to subscriber functions.

Mediator.subscribe options (all are optional; default is empty):

```javascript
{
  predicate: function(*args){ ... }
  priority: 0|1|... 
  calls: 1|2|...
}
```

Predicates return a boolean and are run using whatever args are passed in by the
publishing class. If the boolean is true, the subscriber is run.

Priority marks the order in which a subscriber is called.

`calls` allows you to specify how many times the subscriber is called before it
is automatically removed. This is decremented each time it is called until it
reaches 0 and is removed. If it has a predicate and the predicate does not match,
calls is not decremented.

A Subscriber object is returned when calling Mediator.subscribe. It allows you
to update options on a given subscriber, or to reference it by an id for easy
removal later.

```javascript
{
  id, // guid
  fn, // function
  options, // options
  context, // context for fn to be called within
  channel, // provides a pointer back to its channel
  update(options){ ...} // update the subscriber ({ fn, options, context })
}
```

Examples:

```javascript
var mediator = new Mediator();

// Alert data when the "message" channel is published to
// Subscribe returns a "Subscriber" object
mediator.subscribe("message", function(data){ alert(data); });
mediator.publish("message", "Hello, world");

// Alert the "message" property of the object called when the predicate function returns true (The "From" property is equal to "Jack")
var predicate = function(data){ return data.From === "Jack" };
mediator.subscribe("channel", function(data){ alert(data.Message); }, { predicate: predicate });
mediator.publish("channel", { Message: "Hey!", From: "Jack" }); //alerts
mediator.publish("channel", { Message: "Hey!", From: "Audrey" }); //doesn't alert
```

You can remove events by passing in a channel, or a channel and the
function to remove or subscriber id. If you only pass in a channel,
all subscribers are removed.

```javascript
// removes all methods bound directly to a channel, but not subchannels
mediator.remove("channel");

// unregisters *only* MethodFN, a named function, from "channel"
mediator.remove("channel", MethodFN);
```

You can call the registered functions with the Publish method, which accepts 
an args array:

```javascript
mediator.publish("channel", "argument", "another one", { etc: true });
```

You can namespace your subscribing / removing / publishing as such:

```javascript
mediator.subscribe("application:chat:receiveMessage", function(data){ ... });

// will call parents of the appllication:chat:receiveMessage namespace
// (that is, next it will call all subscribers of application:chat, and then
// application). It will not recursively call subchannels - only direct subscribers.
mediator.publish("application:chat:receiveMessage", "Jack Lawson", "Hey");
```

You can update Subscriber priority:

```javascript
var sub = mediator.subscribe("application:chat", function(data){ ... });
var sub2 = mediator.subscribe("application:chat", function(data){ ... });

// have sub2 executed first
mediator.getChannel("application:chat").setPriority(sub2.id, 0);
```

You can update Subscriber callback, context, and/or options:

```javascript
sub.update({ fn: ..., context: { }, options: { ... });
```

You can stop the chain of execution by calling channel.stopPropagation():

```javascript
// for example, let's not post the message if the from and to are the same
mediator.subscribe("application:chat", function(data, channel){
  alert("Don't send messages to yourself!");
  channel.stopPropagation();
}, options: {
  predicate: function(data){ return data.From == data.To },
  priority: 0
});
```



License
-------
This class and its accompanying README and are 
[MIT licensed](http://www.opensource.org/licenses/mit-license.php).

