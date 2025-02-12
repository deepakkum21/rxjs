# Multicasters [Hot Observables]

## 1. Subject

- A Subject is a `special type of Observable that allows values to be multicasted to many Observers.`
- Subjects are like EventEmitters.
- `Every Subject is an Observable and an Observer.`
- You can `subscribe to a Subject`, and you `can call next to feed values as well as error and complete`.
- can subscribe
- can call next/complete/error notification

## 2. BehaviorSubject

- A `variant of Subject that requires an initial value`.
- it `emits its current value whenever it is subscribed to`.
- The BehaviorSubject has the characteristic that `it stores the “current” value. This means that you can always directly get the last emitted value` from the BehaviorSubject.
- two ways to get this last emitted value

  - `accessing the .value property`
  - `subscribe to it`

         const subject = new BehaviorSubject<number>(Math.random());

         // subscriber 1
         subject.subscribe((data) => {
             console.log('Subscriber A:', data);
         });

         subject.next(Math.random());
         subject.next(Math.random());

         // subscriber 2
         subject.subscribe((data) => {
             console.log('Subscriber B:', data);
         });

         subject.next(Math.random());

         console.log(subject.value);

         // output
         Subscriber A: 0.20824763078070063
         Subscriber A: 0.36220992449789247
         Subscriber A: 0.5907925361687225
         Subscriber B: 0.5907925361687225
         Subscriber A: 0.9258619726220476
         Subscriber B: 0.9258619726220476
         0.9258619726220476

## 3. ReplaySubject

- The ReplaySubject is comparable to the BehaviorSubject in the way that it can send “old” values to new subscribers.
- It however has the `extra characteristic that it can record a part of the observable execution and therefore store multiple old values and “replay” them to new subscribers.`
- When creating the ReplaySubject you `can specify how much values you want to store and for how long you want to store them`. In other words you can specify: “I want to store the last 5 values,

            const replaySubject = new ReplaySubject<number>(3);

            // subscriber 1
            replaySubject.subscribe((data) => {
                console.log('Subscriber A:', data);
            });

            replaySubject.next(Math.random());
            replaySubject.next(Math.random());

            // subscriber 2
            replaySubject.subscribe((data) => {
                console.log('Subscriber B:', data);
            });

            replaySubject.next(Math.random());

            replaySubject.subscribe((data) => {
                console.log('Subscriber C:', data);
            });

            // output
            Subscriber A: 0.22299229984778135
            Subscriber A: 0.9287063154442443
            Subscriber B: 0.22299229984778135
            Subscriber B: 0.9287063154442443
            Subscriber A: 0.566455396668782
            Subscriber B: 0.566455396668782
            Subscriber C: 0.22299229984778135
            Subscriber C: 0.9287063154442443
            Subscriber C: 0.566455396668782

## 4. asyncSubject

- The AsyncSubject is a Subject variant where `only the last value of the Observable execution is sent to its subscribers`, and `only when the execution completes`.

            const asyncSubject = new AsyncSubject<number>();

            // subscriber 1
            asyncSubject.subscribe((data) => {
                console.log('Subscriber A:', data);
            });

            asyncSubject.next(1);
            asyncSubject.next(2);

            // subscriber 2
            asyncSubject.subscribe((data) => {
                console.log('Subscriber B:', data);
            });

            asyncSubject.next(3);

            asyncSubject.subscribe((data) => {
                console.log('Subscriber C:', data);
            });

            asyncSubject.complete(); // when complete is called then only the last emitted value will be sent to all the subscribers

            // output
            Subscriber A: 3
            Subscriber B: 3
            Subscriber C: 3
