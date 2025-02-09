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
                console.log('Teardown');  // this runs atlast when complete or error notification is triggered
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
                console.log('Teardown');  // this runs atlast when complete or error notification is triggered
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
