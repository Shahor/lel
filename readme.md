[![NPM](https://nodei.co/npm/numparser.png?downloads=true)](https://nodei.co/npm/numparser.png?downloads=true)

# Potentially dangerous behavior from Promises

I found about about this after I found out about the problem the hard way, but go [read the spec](https://promisesaplus.com/), and especially the part about thenables and ```The Promise Resolution Procedure```

```“thenable” is an object or function that defines a then method.```

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

I initially found out about this while experimenting with Proxies and thought the issue was with their implementation, but as you can see in the following code (and this [jsbin](http://jsbin.com/reyococeca/edit?js,console)) it is really in the Promise implementation [edit : spec].

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

# Debugging

As you can see, I have been overprotective with the bits of code above, on purpose.
What it shows is that you have *no way* to know where the problem comes from.

The reason for that is the Promise will never resolve nor reject.

A thenable's then function is given the resolve|reject callbacks and should act on it like so : 

```js
let obj = {
	then : (resolve, reject) => {
		resolve(42)
	}
}

Promise.resolve(obj)
	.then(value => {
		console.log(value) // 42
	})
```

Otherwise, you now know what happens.

There's no fix for this, and if someone decides to play bad with this stuff, you're in for a hell of a debugging session. 

Yay js, I guess...

# Warnings 

Please be careful when using this package (because obviously, people will do it. Probably for reasons). 

But you would know that, if you reached this point.
Right? 

¯\\\_(ツ)_/¯


p.s. : Obviously I'm publishing this on npm to troll a little. But this is a serious issue, imo
