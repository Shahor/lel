# Potentially dangerous behavior from Promises

Promises seem to try and find it a value has the method .then before wrapping it to pass it in the resolve/reject callbacks.
Its like duck typing, except in this case its not good enough. 


Here's how I found out about the problem, and what happens exactly
(you can test this code on [jsbin](http://jsbin.com/qizexuredo/edit?js,console))

```js
let proxy = new Proxy({}, {
    get (target, method) {
        return (...args) => {
            console.log(`trying to access property ${method}`)
        }
    }
})

proxy.Hi() // Just to show it works as intended
try {
    console.log("Before promise resolve")
    Promise.resolve(proxy)
        .then(v => {
            console.log("In resolved callback", v)
        })
        .catch(e => {
            console.error("In rejected callback", e)
        })
} catch (e) {
    console.error("In catch block", e)
}
```

# The problem

I initially found out about this while experimenting with Proxies and thought the issue was with their implementation, but as you can see in the following code (and this [jsbin](http://jsbin.com/reyococeca/edit?js,console)) it is really in the Promise implementation, and happens at least on Firefox/Chrome.

```js
let obj = {
  Hi : () => {
    console.log("Hi")
  },
  then : () => {
    console.log("Trolling hard")
  }
}

obj.Hi() // Just to show it works as intended
try {
    console.log("Before promise resolve")
    Promise.resolve(obj)
        .then(v => {
            console.log("In resolved callback", v)
        })
        .catch(e => {
            console.error("In rejected callback", e)
        })
} catch (e) {
    console.error("In catch block", e)
}
```

# The end of the world

So basically if at any point someone introduces the code from this package 

```js
Object.prototype.then = () => {} // Break all the REST apis \o/
```

in a popular one, all hell will break lose.

The same happens, and this could be very painful, if someone manages to hack a CDN and do this as well. 

# Warnings 

Please be careful when using this package (because obviously, people will do it. Probably for reasons). 

But you would know that, if you reached this point.
Right? 

¯\\\_(ツ)_/¯


p.s. : Obviously I'm publishing this on npm to troll a little. But this is a serious issue, imo
