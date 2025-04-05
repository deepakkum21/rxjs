# 1. shareReplay(bufferSize?: number)

- Share source and replay specified number of emissions on subscription.
- we will cache the HTTP response first time. It’s mandatory to store cache value in a variable, for the same observable
```ts
        allUsers$: Observable<any>;
        maleUsers$: Observable<any>;
        femaleUsers$: Observable<any>;

        constructor(private http: HttpClient) {
        }

        ngOnInit(): void {
            /*
            If you call every time this.http.get(API_ENDPOINT).pipe(shareReplay(1)),
            each time http request will be triggered. It will create new observable each time.
            If you want to make http call once and cache data, the following is recommended.
            */
            this.allUsers$ =  this.getAllUsers(); // storing shared observable in local variable

            this.maleUsers$ = this.allUsers$.pipe(
            map((res) => res.filter((user: any) => user.gender === 'male'))
            );
            this.femaleUsers$ = this.allUsers$.pipe(
            map((res) => res.filter((user: any) => user.gender === 'female'))
            );
        }

        getAllUsers(): Observable<any> {
            // shareReplay(1) means only one value will be cache
            return this.http.get('https://raw.githubusercontent.com/piyalidas10/dummy-json/main/fakeuser.json').pipe(shareReplay(1));
        }
```
