# Summary
We have the following Javascript code:
```javascript
const EventEmitter = require('events');

class A extends EventEmitter {
	run() {
		console.log('> A.run()', '|', 'A is running');
		this.emit('done');
	}
}

class B extends EventEmitter {
	run() {
		console.log('> B.run()', '|', 'B is running');
		this.emit('done');
	}
}

const emitterA = new A();
const emitterB = new B();

emitterA.on('done', () => {
	console.log('@ A::done', '|', 'A is done', '[AH1]');
	emitterB.run(); // Line #22
});

emitterA.on('done', () => {
	console.log('@ A::done', '|', 'A is done', '[AH2]');
});

emitterB.on('done', () => {
	console.log('@ B::done', '|', 'B is done', '[BH1]');
	emitterA.run(); // Line #31
});

emitterA.run();
```

Calling `node script.js` will run the code for a short while and then crash with the following message - `RangeError: Maximum call stack size exceeded`.

Here's an output sample:

```
> A.run() | A is running
@ A::done | A is done [AH1]
> B.run() | B is running
@ B::done | B is done [BH1]
> A.run() | A is running
@ A::done | A is done [AH1]
> B.run() | B is running
@ B::done | B is done [BH1]
> A.run() | A is running
...
> A.run() | A is running
@ A::done | A is done [AH1]
node:internal/console/constructor:290
        if (isStackOverflowError(e))
            ^

RangeError: Maximum call stack size exceeded
```

Notice that the second handler is never called.

# Questions
 - Can explain what is going on and why this code fails?
 - By modifying lines `#22` and `#31` (marked above), how would you fix the code so it runs *forever* with the following output?

```
> A.run() | A is running
@ A::done | A is done [AH1]
@ A::done | A is done [AH2]
> B.run() | B is running
@ B::done | B is done [BH1]
> A.run() | A is running
@ A::done | A is done [AH1]
@ A::done | A is done [AH2]
...
> B.run() | B is running
@ B::done | B is done [BH1]
> A.run() | A is running
@ A::done | A is done [AH1]
@ A::done | A is done [AH2]
> B.run() | B is running
@ B::done | B is done [BH1]
...
Infinity
```
*Hint: think about how Node.js schedules callbacks on the event loop!*
