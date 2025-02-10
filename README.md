# rxjs

- provide operators to handle asynchronous code
-

![observable image](./images/observer-observable-subscription.png)

## Subscription lifecycle

![Subscription lifecycle](./images/subscription-lifecycle.png)

## Teardown logic when Observable completes

        const observable$ = new Observable<string>(subscriber => {
            console.log('Observable executed');
            subscriber.next('Alice');
            subscriber.next('Ben');
            setTimeout(() => {
                subscriber.next('Charlie');
                subscriber.complete();
            }, 2000);

            return () => {
                console.log('Teardown');  // this runs atlast when complete/unsubscribes or error notification is triggered
            };
        });

        console.log('Before subscribe');
        observable$.subscribe({
            next: value => console.log(value),
            complete: () => console.log('Completed')
        });
        console.log('After subscribe');

        // output
        Before subscribe
        Observable executed
        Alice
        Ben
        After subscribe
        Charlie
        Completed
        Teardown

## Teardown logic when Observable error out

        const observable$ = new Observable<string>(subscriber => {
            console.log('Observable executed');
            subscriber.next('Alice');
            subscriber.next('Ben');
            setTimeout(() => {
                subscriber.next('Charlie');
            }, 2000);
            setTimeout(() => subscriber.error(new Error('Failure')), 4000);

            return () => {
                console.log('Teardown');  // this runs atlast when complete/unsubscribes or error notification is triggered
            };
        });

        console.log('Before subscribe');
        observable$.subscribe({
            next: value => console.log(value),
            error: err => console.log(err.message),
            complete: () => console.log('Completed')
        });
        console.log('After subscribe');

        //
        Before subscribe
        Observable executed
        Alice
        Ben
        After subscribe
        Charlie
        Failure
        Teardown

## Unsubscribe & teardown importance

        const interval$ = new Observable<number>(subscriber => {
            let counter = 1;

            const intervalId = setInterval(() => {
                console.log('Emitted', counter);
                subscriber.next(counter++);
            }, 2000);

            return () => {
                clearInterval(intervalId);  // if teardown is not written to clear the setInterval, the observable would not able
                                            // emitted any value but setInterval logs would have still printed
            };
        });

        const subscription = interval$.subscribe(value => console.log(value));

        setTimeout(() => {
            console.log('Unsubscribe');
            subscription.unsubscribe();
        }, 7000);


        // output without teardown
        Emitted 1
        1
        Emitted 2
        2
        Emitted 3
        3
        Unsubscribe
        Emitted 4
        Emitted 5
        Emitted 6
        so on...

        // output with teardown
        Emitted 1
        1
        Emitted 2
        2
        Emitted 3
        3
        Unsubscribe

## HOT vs COLD Observable

| Hot Observable                        | Cold Observable                             |
| ------------------------------------- | ------------------------------------------- |
| Multicast the data from common source | Produces the data inside                    |
| All subscribers common data           | new Subscriber will have new data           |
| eg:- DOM event, State, Subject        | set of values, HTTP request, Timer/Interval |

# Creation Function

## 1. of

- `Converts the arguments to an observable sequence`.
- Each argument becomes a next notification

        of(10,20,30).subscribe(val => console.log(val))
        output
        10
        20
        30

        of([10,20,30]).subscribe(val => console.log(val))
        output
        [10,20,30]

- `polyfill of operator`

        function ourOwnOf(...args: string[]): Observable<string> {
            return new Observable<string>(subscriber => {
            for(let i = 0; i < args.length; i++) {
                subscriber.next(args[i]);
            }
                subscriber.complete();
            });
        }

## 2. from

- `Creates an Observable from an Array, an array-like object,`
  - array
  - a Promise,
  - an iterable object,
  - an Observable-like object.
- Converts almost anything to an Observable

        from([10,20,30]).subscribe(val => console.log(val))
        output
        10
        20
        30

## 3. fromEvent

- Creates an Observable that emits events of a specific type coming from the `given event target`

        const triggerButton = document.querySelector('button#trigger');

        const subscription = fromEvent<MouseEvent>(triggerButton, 'click').subscribe(
            event => console.log(event.type, event.x, event.y)
        );

- polyfill for fromEvent

        const triggerClick$ = new Observable<MouseEvent>(subscriber => {
            const clickHandlerFn = event => {
                console.log('Event callback executed');
                subscriber.next(event);
            };

            triggerButton.addEventListener('click', clickHandlerFn);

            // super imp as unsubscribing doesn't clear the click event
            // teardown will be run when unsubscribe/complete/error notification is triggered
            return () => {
                triggerButton.removeEventListener('click', clickHandlerFn);
            };
        });

        const subscription = triggerClick$.subscribe(
            event => console.log(event.type, event.x, event.y)
        );

        setTimeout(() => {
            console.log('Unsubscribe');
            subscription.unsubscribe();
        }, 5000);

## 4. timer

- internally works same same setTimer
- unsubscribing leads to cleaning of all events, intervals all cleanup

        const timerRxjs$ = timer(200).subscribe({
            next: (value) => console.log(value),
            complete: () => console.log(' timer rxjs complete'),
        });

        setTimeout(() => timerRxjs$.unsubscribe(), 3000);

- polyfill for timer

        const timer$ = new Observable<number>((subscriber) => {
            const timeoutId = setTimeout(() => {
                console.log('Timeout!');
                subscriber.next(0);
                subscriber.complete();
            }, 2000);

            // super imp as unsubscribing doesn't clear the click event
            // teardown will be run when unsubscribe/complete/error notification is triggered
            return () => clearTimeout(timeoutId);
        });

        const subscription = timer$.subscribe({
            next: (value) => console.log(value),
            complete: () => console.log('Completed'),
        });

        setTimeout(() => {
            subscription.unsubscribe();
            console.log('Unsubscribe');
        }, 1000);

## 4. interval

- internally works same same setInterval
- unsubscribing leads to cleaning of all events, intervals all cleanup

        const intervalRxjs$ = interval(1000).subscribe({
            next: (value) => console.log(value++),
            complete: () => console.log(' interval rxjs complete'),
        });

        setTimeout(() => intervalRxjs$.unsubscribe(), 3000);

- polyfill for interval

        const interval$ = new Observable<number>((subscriber) => {
            let counter = 0;

            const intervalId = setInterval(() => {
                console.log('Timeout!');
                subscriber.next(counter++);
            }, 1000);

            // super imp as unsubscribing doesn't clear the click event
            // teardown will be run when unsubscribe/complete/error notification is triggered
            return () => clearInterval(intervalId);
        });

        const subscription = interval$.subscribe({
            next: (value) => console.log(value),
            complete: () => console.log('Completed'),

        });

        setTimeout(() => {
            subscription.unsubscribe();
            console.log('Unsubscribe');
        }, 5000);

## 5. forkJoin(...args: any[]): Observable<any>

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

## 6. combineLatest

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
