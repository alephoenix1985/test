## NodeJs
### Exercice: Is there a problem? (1 points)

```
// Call web service and return count user, (got is library to call url)
async function getCountUsers() {
return { total: await got.get('https://my-webservice.moveecar.com/users/count') };
}

// Add total from service with 20
async function computeResult() {
const result = getCountUsers();
return result.total + 20;
}
```
R: at function  `getCountUsers` first line, is trying to get values from a promise function so it wont return expected number, it requires to add `await`

const result = `await` getCountUsers()

### Exercice: Is there a problem? (2 points)

```
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
    return await got.get('https://my-webservice.moveecar.com/vehicles/total');
}

function getPlurial() {
    let total;
    getTotalVehicles().then(r => total = r);
    if (total <= 0) {
        return 'none';
    }
    if (total <= 10) {
        return 'few';
    }
    return 'many';
}

```
R: `getPlurial` is running a promise function `getTotalVehicles` which will not resolve the "then" function that sets the total only until the code has finished, so it goes ahead and passes conditions with `total=null` and will return "many" as default.

### Exercice: Unit test (2 points)

```
import { expect, test } from '@jest/globals';

function getCapitalizeFirstWord(name: string): string {
if (name == null) {
throw new Error('Failed to capitalize first word with null');
}
if (!name) {
return name;
}
return name.split(' ').map(
n => n.length > 1 ? (n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()) : n
).join(' ');
}

test('1. test', async function () {
...
});
```
R:
```
import { expect, test } from '@jest/globals';

function getCapitalizeFirstWord(name: string): string {
if (name == null) {
throw new Error('Failed to capitalize first word with null');
}
if (!name) {
return name;
}
return name.split(' ').map(
n => n.length > 1 ? (n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()) : n
).join(' ');
}

test('1. test', async function () {
    expect(getCapitalizeFirstWord('ASD QWE')).toBe('Asd Qwe')
});
```

## Angular
### Exercice: Is there a problem and improve the code (5 points)

```
@Component({
  selector: 'app-users',
  template: `
    <input type="text" [(ngModel)]="query" (ngModelChange)="querySubject.next($event)">
    <div *ngFor="let user of users">
        {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit {

  query = '';
  querySubject = new Subject<string>();

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }

  ngOnInit(): void {
    concat(
      of(this.query),
      this.querySubject.asObservable()
    ).pipe(
      concatMap(q =>
        timer(0, 60000).pipe(
          this.userService.findUsers(q)
        )
      )
    ).subscribe({
      next: (res) => this.users = res
    });
  }
}
```
R: this will refresh the search after 600000ms after change the input value
```
@Component({
  selector: 'app-users',
  template: `
    <input type="text" #query (input)="querySubject.next(query.value)">
    <div *ngFor="let user of users">
      {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit {
  querySubject = new Subject<string>();

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }

  ngOnInit(): void {
    this.querySubject.pipe(
        switchMap(d => timer(0, 60000).pipe(
          map(_ => this.userService.findUsers(d))
        ))
    ).subscribe({
      next: (res: any) => {
        this.users = res
      }
    });
  }
}
```

### Exercice: Improve performance (5 points)
```
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users">
        {{ getCapitalizeFirstWord(user.name) }}
    </div>
  `
})
export class AppUsers {

  @Input()
  users: { name: string; }[];

  constructor() {}
  
  getCapitalizeFirstWord(name: string): string {
    return name.split(' ').map(n => n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()).join(' ');
  }
}
```
R:
```
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users">
      {{ getCapitalizeFirstWord(user.name) }}
    </div>
  `
})
export class AppUsers {

  @Input()
  users: {
    name: string;
  }[] = [];

  constructor() {
  }


  getCapitalizeFirstWord(name: string): string {
    return name.replace(/(^[a-z]|\s[a-z])/g, text => text.toUpperCase())
  }
}
```
### Exercice: Forms (8 points)
```
{
  email: string; // mandatory, must be a email
  name: string; // mandatory, max 128 characters
  birthday?: Date; // Not mandatory, must be less than today
  address: { // mandatory
    zip: number; // mandatory
    city: string; // mandatory, must contains only alpha uppercase and lower and space
  };
}
```
```
@Component({
  selector: 'app-user-form',
  template: `
    <form>
        <input type="text" placeholder="email">
        <input type="text" placeholder="name">
        <input type="date" placeholder="birthday">
        <input type="number" placeholder="zip">
        <input type="text" placeholder="city">
    </form>
  `
})
export class AppUserForm {

  @Output()
  event = new EventEmitter<{ email: string; name: string; birthday: Date; address: { zip: number; city: string; };}>;
  
  constructor(
    private formBuilder: FormBuilder
  ) {
  }

  doSubmit(): void {
    this.event.emit(...);
  }
}
```
R:
```
@Component({
  selector: 'app-user-form',
  template: `
    <form [formGroup]="form" (ngSubmit)="doSubmit()">
      <input type="text" formControlName="email" placeholder="email">
      <input type="text" formControlName="name" placeholder="name">
      <input type="date" formControlName="birthday" placeholder="birthday">
      <div formGroupName="address">
        <input type="number" formControlName="zip" placeholder="zip">
        <input type="text" formControlName="city" placeholder="city">
      </div>
      <button type="submit" [disabled]="!form.valid">Submit</button>
    </form>
  `
})
export class AppUserForm {
  @Output()
  event = new EventEmitter<{
    email: string;
    name: string;
    birthday: Date;
    address: {
      zip: number;
      city: string;
    };
  }>;
  form = this.formBuilder.group({
    email: ['', [Validators.required, Validators.email]],
    name: ['', [Validators.required, Validators.maxLength(128)]],
    birthday: [null, [this.dateValidator]],
    address: this.formBuilder.group({
      zip: [null, [Validators.required]],
      city: ['', [Validators.required, Validators.pattern('^[a-zA-Z0-9]+$')]]
    })
  });

  constructor(
    private formBuilder: FormBuilder
  ) {
  }


  dateValidator(control) {

    const date = control.value;
    if (date) {
      const isDateBeforeToday = new Date(date) < new Date();
      if (!isDateBeforeToday) {
        return {'invalidDate': true}
      }
    }
    return null;
  }

  doSubmit(): void {
    if (this.form.value) {
      const v = this.form.getRawValue();
      this.event.emit({
        email: v.email ?? '',
        name: v.name ?? '',
        birthday: v.birthday ?? new Date(),
        address: {
          zip: Number(v.address?.zip),
          city: v.address?.city ?? ''
        }
      });
    }
  }
}
```
## CSS & Bootstrap
### Exercice: Card (5 points)
![](https://gitlab.com/gabriel-allaigre/exercices/-/wikis/uploads/0388377207d10f8732e1d64623a255b6/image.png)
R:
```
<div class="card w-100 position-relative">
    <div class="position-absolute top-0 end-0">
      <span class="rounded-pill position-absolute translate-middle text-white fw-bold px-2 bg-danger">+123</span>
    </div>
    <div class="card-body">
      <h5 class="card-title">Exercice</h5>
      <p class="card-text">Redo this card in css and if possible using bootstrap 4 or 5</p>
      <div class="d-flex gap-2 w-100 justify-content-end">
        <div class="btn-group">
          <span class="btn bg-black text-white">Got it</span>
          <span class="btn border-1 border-black">I don't know</span>
        </div>
        <button type="button" class="btn btn-secondary d-flex gap-2 align-items-center">
          Options
          <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12s" fill="currentColor"
               class="bi bi-caret-down-fill" viewBox="0 0 16 16">
            <path
              d="M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z"/>
          </svg>
        </button>
      </div>
    </div>
  </div>
```
## MongoDb
### Exercice: MongoDb request (3 points)
```
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```

R1:
```
db.collections('users').aggregate(
 {
  $and: [{
    $or: [{
      email: "TEXT"
    }, {
      first_name: /^TEXT/
    }, {
      last_name: /TEXT$/
    }]
  }, {
    last_connection_date: {
      $gt: ISODate(new Date(new Date().setMonth(new Date().getMonth() - 6)).toISOString())
    }
  }]
}
);
```
R2: creating indexes

### Exercice: MongoDb aggregate (5 points)

```
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```
R:
```
db.collections('users').aggregate(
[
  {
    $group: {
      _id: "role",
      users: { $push: "$email" },
    },
  },
]
)
```
### Exercice: MongoDb aggregate (5 points)
```
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
    addresses: {
        zip: number;
        city: string;
    }[]:
  }
```
R1:
```
  db.collections('users').updateOne(
  {
    _id: ObjectId("5cd96d3ed5d3e20029627d4a")
  },
  {
    $set: {
        last_connection_date: "$$NOW"
    }
  }
  );
```
R2:
```
  db.collections('users').updateOne(
  {
    _id: ObjectId("5cd96d3ed5d3e20029627d4a")
  },
  {
    $push: {
        roles: "admin"
    }
  }
  );
```
R3:
```
  db.collections('users').updateOne(
  {
    _id: ObjectId("5cd96d3ed5d3e20029627d4a")
  },
  {
    addresses.$[a].city': "Paris 1"
  },
  {
    arrayFilters: [{'a.zip': 75001}]
  }
 );
```
