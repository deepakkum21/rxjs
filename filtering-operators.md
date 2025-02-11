# Filtering operators

## 1. first(predicate?, defaultValue?: any)

- `Emits only the first value (or the first value that meets some condition) emitted by the source Observable.`
- Emits only the first value. Or emits only the first value that passes some test.

            const clicks = fromEvent(document, 'click');
            const result = clicks.pipe(first());
            result.subscribe(x => console.log(x));

            const clicks = fromEvent(document, 'click');
            const result = clicks.pipe(first(ev => (<HTMLElement>ev.target).tagName === 'DIV'));
            result.subscribe(x => console.log(x));

![first](./images/first.png)

## 2. last(predicate?, defaultValue?: any)

- Returns an `Observable that emits only the last item emitted by the source Observable`.
- It `optionally takes a predicate function as a parameter, in which case, rather than emitting the last item from the source Observable`, the resulting Observable will emit the last item from the source Observable that satisfies the predicate.
- It will emit an error notification if the source completes without notification or one that matches the predicate.
- It returns the last value or if a predicate is provided last value that matches the predicate.
- It returns the given default value if no notification is emitted or matches the predicate.

            const source = from(['x', 'y', 'z']);
            const result = source.pipe(last());
            result.subscribe(value => console.log(`Last alphabet: ${ value }`));
            // Outputs
            // Last alphabet: z

            const source = from(['x', 'y', 'z']);
            const result = source.pipe(last(char => char === 'a', 'not found'));
            result.subscribe(value => console.log(`'a' is ${ value }.`));
            // Outputs
            // 'a' is not found.

![last](./images/last.png)
