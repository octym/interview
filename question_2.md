# Node.js - Question #2
We will be using the code from question 1 but this time we are running the emitters inside `setImmediate()`.

*Before:*
`emitter.run();`

*After:*
`setImmediate(() => {
	emitter.run();
});`

Such that the new code looks like this:
```javascript
import EventEmitter from 'events';

const $MAX_RUNS = 2;
let $COUNTER = 0;

class A extends EventEmitter {
	run() {
		console.log('> A.run()', '|', 'A is running');
		this.emit('done');
	}
}

class B extends EventEmitter {
	run() {
		console.log('> B.run()', '|', 'B is running');
		if (++$COUNTER < $MAX_RUNS) {
			this.emit('done');
		} else {
			console.log('---------', '|', 'FINISHED WITH B', $MAX_RUNS);
		}
	}
}

const emitterA = new A();
const emitterB = new B();

emitterA.on('done', () => {
	console.log('@ A::done', '|', 'A is done', '[AH1]');
	setImmediate(() => {
		emitterB.run();
	});
});

emitterA.on('done', () => {
	console.log('@ A::done', '|', 'A is done', '[AH2]');
});

emitterB.on('done', () => {
	console.log('@ B::done', '|', 'B is done', '[BH1]');
	setImmediate(() => {
		emitterA.run();
	});
});

emitterA.run();
```

Which of these two outputs is the correct one?

## Output A
```javascript
> A.run() | A is running                             | LOOP_COUNTER = 0
@ A::done | A is done running [AH1]                  | LOOP_COUNTER = 0
@ A::done | A is done running [AH2]                  | LOOP_COUNTER = 0

> B.run() | B is running                             | LOOP_COUNTER = 0
@ B::done | B is done running [BH1]                  | LOOP_COUNTER = 1

> A.run() | A is running                             | LOOP_COUNTER = 1
@ A::done | A is done running [AH1]                  | LOOP_COUNTER = 1
@ A::done | A is done running [AH2]                  | LOOP_COUNTER = 1

> B.run() | B is running                             | LOOP_COUNTER = 1
--------- | FINISHED WITH B                          | LOOP_COUNTER = 2
```

## Output B
```javascript
> A.run() | A is running                             | LOOP_COUNTER = 0
@ A::done | A is done running [AH1]                  | LOOP_COUNTER = 0

> B.run() | B is running                             | LOOP_COUNTER = 0
@ B::done | B is done running [BH1]                  | LOOP_COUNTER = 1

> A.run() | A is running                             | LOOP_COUNTER = 1
@ A::done | A is done running [AH1]                  | LOOP_COUNTER = 1

> B.run() | B is running                             | LOOP_COUNTER = 1
--------- | FINISHED WITH B                          | LOOP_COUNTER = 2

@ A::done | A is done running [AH2]                  | LOOP_COUNTER = 2
@ A::done | A is done running [AH2]                  | LOOP_COUNTER = 2
```
