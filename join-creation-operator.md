# Join Creation Operators

## 1. forkJoin(...args: any[]): Observable<any>

- `Accepts an Array of ObservableInput or a dictionary Object of ObservableInput` and
- returns an Observable that emits either an array of values in the exact same order as the passed array, or a dictionary of values in the same shape as the passed dictionary.
- `Wait for Observables to complete and then combine last values they emitted; complete immediately if an empty array is passed.`

        const randomName$ = ajax('https://random-data-api.com/api/name/random_name');
        const randomNation$ = ajax('https://random-data-api.com/api/nation/random_nation');
        const randomFood$ = ajax('https://random-data-api.com/api/food/random_food');

        forkJoin([randomName$, randomNation$, randomFood$]).subscribe(
            ([nameAjax, nationAjax, foodAjax]) => console.log(`${nameAjax.response.first_name} is from ${nationAjax.response.capital} and likes to eat ${foodAjax.response.dish}.`)
        );

- `if any of the observable passed errors out, forkJoin will not wait for other subs to complete.`

        const a$ = new Observable(subscriber => {
            setTimeout(() => {
                subscriber.next('A');
                subscriber.complete();
            }, 5000);

            return () => {
                console.log('A teardown');
            };
        });

        const b$ = new Observable(subscriber => {
            setTimeout(() => {
                subscriber.error('Failure!');
            }, 3000);

            return () => {
                console.log('B teardown');
            };
        });

        forkJoin([a$, b$]).subscribe({
            next: value => console.log(value),
            error: err => console.log('Error:', err)
        });

![forkJoin](./images/forkJoin.png)

## 2. combineLatest

- Whenever any input Observable emits a value, it computes a formula using the latest values from all the inputs, then emits the output of that formula.
- it will wait for all the observables to emit first set of data, and on subsequent data emitted by any of the observable it will combine with the last emitted value fo the other observable, this will continue will all observables gets completed.

![combineLatest](./images/combineLatest.png)

        const firstTimer = timer(0, 1000); // emit 0, 1, 2... after every second, starting from now
        const secondTimer = timer(500, 1000); // emit 0, 1, 2... after every second, starting 0,5s from now
        const combinedTimers = combineLatest([firstTimer, secondTimer]);
        combinedTimers.subscribe(value => console.log(value));
        // Logs
        // [0, 0] after 0.5s
        // [1, 0] after 1s
        // [1, 1] after 1.5s
        // [2, 1] after 2s

- `same as other rxjs, if any of the observable results in error, the combineLatest will produce error, stopping all observables.`

## 3. concat(...args)

- Creates an output Observable which `sequentially emits all values from the first given Observable and then moves on to the next.`
- Concatenates multiple Observables together by sequentially emitting their values, one Observable after the other.
- concat will subscribe to first input Observable and emit all its values, without changing or affecting them in any way. When that Observable completes, it will subscribe to then next Observable passed and, again, emit its values. This will be repeated, until the operator runs out of Observables. When last input Observable completes, concat will complete as well. At any given moment only one Observable passed to operator emits values

![concat](./images/concat.png)

    const timer = interval(1000).pipe(take(4));
    const sequence = range(1, 10);
    const result = concat(timer, sequence);
    result.subscribe(x => console.log(x));

    // results in:
    // 0 -1000ms-> 1 -1000ms-> 2 -1000ms-> 3 -immediate-> 1 ... 10

## 4. merge(...args)

- Creates an output Observable which `concurrently emits all values from every given input Observable.`
- Flattens multiple Observables together by blending their values into one Observable.
- merge subscribes to each given input Observable (as arguments), and simply forwards (without doing any transformation) all the values from all the input Observables to the output Observable. The output Observable only completes once all input Observables have completed. Any error delivered by an input Observable will be immediately emitted on the output Observable.

![merge](./images/merge.png)

        const clicks = fromEvent(document, 'click');
        const timer = interval(1000);
        const clicksOrTimer = merge(clicks, timer);
        clicksOrTimer.subscribe(x => console.log(x));

        // Results in the following:
        // timer will emit ascending values, one every second(1000ms) to console
        // clicks logs MouseEvents to console every time the "document" is clicked
        // Since the two streams are merged you see these happening
        // as they occur.

## 5. partition(source: Observable, predicate)

- `Splits the source Observable into two, one with values that satisfy a predicate, and another with values that don't satisfy the predicate.`
- It's like filter, but returns two Observables: one like the output of filter, and the other with values that did not pass the condition.
- partition outputs an array with two Observables that partition the values from the source Observable through the given predicate function. The first Observable in that array emits source values for which the predicate argument returns true. The second Observable emits source values for which the predicate returns false. The first behaves like filter and the second behaves like filter with the predicate negated.

          const observableValues = of(1, 2, 3, 4, 5, 6);
          const [evens$, odds$] = partition(observableValues, value => value % 2 === 0);

          odds$.subscribe(x => console.log('odds', x));
                  evens$.subscribe(x => console.log('evens', x));

          // Logs:
          // odds 1
          // odds 3
          // odds 5
          // evens 2
          // evens 4
          // evens 6

  ![partition](./images/partition.png)

## 6. race(...observables)

- its like a race where the from the list of observable which emits the first will win and will continue till it completes, others are unsubscribed as soon as first will emit a value.
- race returns an observable, that when subscribed to, subscribes to all source observables immediately. `As soon as one of the source observables emits a value, the result unsubscribes from the other sources`. The resulting observable will forward all notifications, including error and completion, from the "winning" source observable.
- `If one of the used source observable throws an errors before a first notification the race operator will also throw an error, no matter if another source observable could potentially win the race`.
- race can be useful for selecting the response from the fastest network connection for HTTP or WebSockets. race can also be useful for switching observable context based on user input

            const obs1 = interval(7000).pipe(map(() => 'slow one'));
            const obs2 = interval(3000).pipe(map(() => 'fast one'));
            const obs3 = interval(5000).pipe(map(() => 'medium one'));

            race(obs1, obs2, obs3)
            .subscribe(winner => console.log(winner));

            // Outputs
            // a series of 'fast one'

![race](./images/race.png)
