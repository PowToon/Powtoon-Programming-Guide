
# PowToon clean code guide

*A mostly reasonable approach to writing a clean code based on [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)*

## Table of Contents
  1. [Introduction](#introduction)
  1. [Variables](#variables)
  1. [Functional Programming](#functional-programming)
  1. [Functions](#functions)
  1. [Objects and Data Structures](#objects-and-data-structures)
  1. [Classes](#classes)
  1. [Testing](#testing)
  1. [Concurrency](#concurrency)
  1. [Error Handling](#error-handling)
  1. [Comments](#comments)

## Introduction
![Humorous image of software quality estimation as a count of how many expletives
you shout when reading code](http://www.osnews.com/images/comics/wtfm.jpg)

Software engineering principles, from Robert C. Martin's book
[*Clean Code*](httpse//www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for JavaScript. This is not a style guide. It's a guide to producing
readable, reusable, and refactorable software in JavaScript.

Not every principle herein has to be strictly followed, and even fewer will be
universally agreed upon. These are guidelines and nothing more, but they are
ones codified over many years of collective experience by the authors of
*Clean Code*.

Our craft of software engineering is just a bit over 50 years old, and we are
still learning a lot. When software architecture is as old as architecture
itself, maybe then we will have harder rules to follow. For now, let these
guidelines serve as a touchstone by which to assess the quality of the
JavaScript code that you and your team produce.

## **Variables**
### Use meaningful and pronounceable variable names

**Bad:**
```javascript
const yyyymmdstr = moment().format('YYYY/MM/DD')
```

**Good:**
```javascript
const currentDate = moment().format('YYYY/MM/DD')
```
**[⬆ back to top](#table-of-contents)**

### Use the same vocabulary for the same type of variable

**Bad:**
```javascript
getUserInfo()
getClientData()
getCustomerRecord()
```

**Good:**
```javascript
getUser()
```
**[⬆ back to top](#table-of-contents)**

### Use searchable names
We will read more code than we will ever write. It's important that the code we
do write is readable and searchable. By *not* naming variables that end up
being meaningful for understanding our program, we hurt our readers.
Make your names searchable.

**Bad:**
```javascript
// What the heck is 86400000 for?
setTimeout(blastOff, 86400000)
```

**Good:**
```javascript
// Declare them as capitalized `const`.
const MILLISECONDS_IN_A_DAY = 86400000
const SPACE_KEY = 32

setTimeout(blastOff, MILLISECONDS_IN_A_DAY)
```
If a constant is expected to be used in more then one place,
export it to `common/consts`.

**Bad:**
```javascript
// What key has been pressed?
if(keyBoardEvent.code == 32){
  return
}
```

**Good:**
```javascript
import {SPACE_KEY_CODE} from 'common/consts'

if(keyBoardEvent.code == SPACE_KEY_CODE){
  return
}
```

**[⬆ back to top](#table-of-contents)**

### Use explanatory variables
**Bad:**
```javascript
const address = 'One Infinite Loop, Cupertino 95014'
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/
saveCityZipCode(address.match(cityZipCodeRegex)[1], address.match(cityZipCodeRegex)[2])
```

**Good:**
```javascript
const address = 'One Infinite Loop, Cupertino 95014'
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/
const [, city, zipCode] = address.match(cityZipCodeRegex) || []
saveCityZipCode(city, zipCode)
```
**[⬆ back to top](#table-of-contents)**

### Avoid Mental Mapping
Explicit is better than implicit.

**Bad:**
```javascript
const locations = ['Austin', 'New York', 'San Francisco']
locations.forEach((l) => {
  doStuff()
  doSomeOtherStuff()
  // ...
  // ...
  // ...
  // Wait, what is `l` for again?
  dispatch(l)
})
```

**Good:**
```javascript
const locations = ['Austin', 'New York', 'San Francisco']
locations.forEach((location) => {
  doStuff()
  doSomeOtherStuff()
  // ...
  // ...
  // ...
  dispatch(location)
})
```
**[⬆ back to top](#table-of-contents)**

### Don't add unneeded context
If your class/object name tells you something, don't repeat that in your
variable name.

**Bad:**
```javascript
const Car = {
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue'
}

function paintCar(car) {
  car.carColor = 'Red'
}
```

**Good:**
```javascript
const Car = {
  make: 'Honda',
  model: 'Accord',
  color: 'Blue'
}

function paintCar(car) {
  car.color = 'Red'
}
```
**[⬆ back to top](#table-of-contents)**

### Use default arguments instead of short circuiting or conditionals
Default arguments are often cleaner than short circuiting. Be aware that if you
use them, your function will only provide default values for `undefined`
arguments. Other "falsy" values such as `''`, `""`, `false`, `null`, `0`, and
`NaN`, will not be replaced by a default value.

**Bad:**
```javascript
function createMicrobrewery(name) {
  const breweryName = name || 'Hipster Brew Co.'
  // ...
}
```

**Good:**
```javascript
function createMicrobrewery(breweryName = 'Hipster Brew Co.') {
  // ...
}
```
**[⬆ back to top](#table-of-contents)**

### Try to avoid reassignments
Reassignments are less readable and can be slower and potentially buggy because code
be added betweet the reassignment in the future.

**Bad:**
```javascript
const employeesDataToRender = employees.map(employee => {
  const expectedSalary = calculateExpectedSalary(employee)
  const experience = getExperience(employee)

  let portfolio = getGithubLink(employee)

  if (employee.type === 'manager') {
    portfolio = getMBAProjects(employee)
  }

  return {
    expectedSalary,
    experience,
    portfolio
  }
})
```

**Good:**
```javascript
const employeesDataToRender = employees.map(employee => {
  const expectedSalary = calculateExpectedSalary(employee)
  const experience = getExperience(employee)

  const portfolio = employee.type === 'manager' ?
    getMBAProjects(employee) :
    getGithubLink(employee)

  return {
    expectedSalary,
    experience,
    portfolio
  }
})
```

**Better:**
```javascript
function getPortofolio(employee){
  return employee.type === 'manager' ?
    getMBAProjects(employee) :
    getGithubLink(employee)
}

const employeesDataToRender = employees.map(employee => ({
  expectedSalary: calculateExpectedSalary(employee),
  experience: getExperience(employee),
  portfolio: getPortofolio(employee)
}))
```

**[⬆ back to top](#table-of-contents)**

## **Functional Programming**
Functional programming is cleaner, easier to test and works better
with react and redux. Favor this style of programming when you can.
Read more about Functional Programming, the reasoning behind
it and it's benifits here:

* [JavaScript and Functional Programming](https://bethallchurch.github.io/JavaScript-and-Functional-Programming/)

* [How functional programming is different from Object Oriented Programming](http://www.codenewbie.org/blogs/object-oriented-programming-vs-functional-programming)

### Favor functional programming over imperative programming
For the array:
```javascript
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  },
  {
    name: 'Suzie Q',
    linesOfCode: 1500
  },
  {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  },
  {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
]
```

**Bad:**
```javascript
let totalOutput = 0

for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode
}
```

**Good:**
```javascript
const INITIAL_LINES_OF_CODE = 0

const totalOutput = programmerOutput
  .map((programmer) => programmer.linesOfCode)
  .reduce((acc, linesOfCode) => acc + linesOfCode, INITIAL_LINES_OF_CODE)
```
**[⬆ back to top](#table-of-contents)**

### Use Functional Programming instead of OOP
Focus on purity of functions and immutability of objects
and you will find yourself also avoiding OOP.

**Bad:**
```javascript
function Employee({type, basicSalary, experience, MBAProjects, githubLink}){
  //...
}

Employee.prototype.raiseBasicSalary = function(raise){
  this.basicSalary = this.basicSalary + raise //This is not pure.
}

const someEmployees = [
  new Employee({type: 'programmer', basicSalary: 100, experience: 4, MBAProjects: [], githubLink: ''}),
  new Employee({type: 'manager', basicSalary: 200, experience: 8, MBAProjects: [], githubLink: ''}),
  //...
]

someEmployees[0].raiseBasicSalary(10) //This is not pure.

render(someEmployees)
```

**Good:**
```javascript
const getEmployeeWithRaisedSalary = (employee, raise) => {
  return {...employee, basicSalary: employee + raise}
}

let someEmployees = [
  {type: 'programmer', basicSalary: 100, experience: 4, MBAProjects: [], githubLink: ''},
  {type: 'manager', basicSalary: 200, experience: 8, MBAProjects: [], githubLink: ''},
  //...
]

const employeeWithRaisedSalary = getEmployeeWithRaisedSalary(someEmployees[0], 10)
const employeesAfterRaise = [
  employeeWithRaisedSalary,
  ...(someEmployees.slice(1))
]

render(employeesAfterRaise)
```
**[⬆ back to top](#table-of-contents)**

### Avoid Side Effects (part 1)
A function produces a side effect if it does anything other than take a value in
and return another value or values. A side effect could be writing to a file,
modifying some global variable, or accidentally wiring all your money to a
stranger.

Now, you do need to have side effects in a program on occasion. Like the previous
example, you might need to write to a file. What you want to do is to
centralize where you are doing this. Don't have several functions and classes
that write to a particular file. Have one service that does it. One and only one.

The main point is to avoid common pitfalls like sharing state between objects
without any structure, using mutable data types that can be written to by anything,
and not centralizing where your side effects occur. If you can do this, you will
be happier than the vast majority of other programmers.

**Bad:**
```javascript
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
let name = 'Ryan McDermott'

function splitIntoFirstAndLastName() {
  name = name.split(' ')
}

splitIntoFirstAndLastName()

console.log(name) // ['Ryan', 'McDermott']
```

**Good:**
```javascript
function splitIntoFirstAndLastName(name) {
  return name.split(' ')
}

const name = 'Ryan McDermott'
const splittedName = splitIntoFirstAndLastName(name)

console.log(name) // 'Ryan McDermott'
console.log(splittedName) // ['Ryan', 'McDermott']
```
**[⬆ back to top](#table-of-contents)**

### Avoid Side Effects (part 2)
In JavaScript, primitives are passed by value and objects/arrays are passed by
reference. In the case of objects and arrays, if your function makes a change
in a shopping cart array, for example, by adding an item to purchase,
then any other function that uses that `cart` array will be affected by this
addition. That may be great, however it can be bad too. Let's imagine a bad
situation:

The user clicks the "Purchase", button which calls a `purchase` function that
spawns a network request and sends the `cart` array to the server. Because
of a bad network connection, the `purchase` function has to keep retrying the
request. Now, what if in the meantime the user accidentally clicks "Add to Cart"
button on an item they don't actually want before the network request begins?
If that happens and the network request begins, then that purchase function
will send the accidentally added item because it has a reference to a shopping
cart array that the `addItemToCart` function modified by adding an unwanted
item.

A great solution would be for the `addItemToCart` to always clone the `cart`,
edit it, and return the clone. This ensures that no other functions that are
holding onto a reference of the shopping cart will be affected by any changes.

Two caveats to mention to this approach:
  1. There might be cases where you actually want to modify the input object,
but when you adopt this programming practice you will find that those case
are pretty rare. Most things can be refactored to have no side effects!

  2. Cloning big objects can be very expensive in terms of performance. Luckily,
this isn't a big issue in practice because there are
[great libraries](https://facebook.github.io/immutable-js/) that allow
this kind of programming approach to be fast and not as memory intensive as
it would be for you to manually clone objects and arrays.

**Bad:**
```javascript
const addItemToCart = (cart, item) => {
  cart.push({ item, date: Date.now() })
}
```

**Good:**
```javascript
const addItemToCart = (cart, item) => {
  return [...cart, { item, date : Date.now() }]
}
```
**[⬆ back to top](#table-of-contents)**

## **Functions**
### When possible use lodash.

You can always assume everybody knows and uses [all lodash functions](https://lodash.com/docs).

**Bad:**
```javascript
const arr1 = ['a', 'b']
const arr2 = [1, 2]

const zipped = {}
const numberOfKeys = Math.max(arr1.length, arr2.length)
for(let i = 0; i < numberOfKeys; i++){
  zipped[arr1[i]] = arr2[i]
}
console.log(zipped) //{ 'a': 1, 'b': 2 }
```

**Good:**
```javascript
const arr1 = ['a', 'b']
const arr2 = [1, 2]

const zipped = _.zipObject(arr1, arr2)
console.log(zipped) //{ 'a': 1, 'b': 2 }
```
**[⬆ back to top](#table-of-contents)**

### Function arguments (2 or fewer ideally)
Limiting the amount of function parameters is incredibly important because it
makes testing your function easier. Having more than three leads to a
combinatorial explosion where you have to test tons of different cases with
each separate argument.

One or two arguments is the ideal case, and three should be avoided if possible.
Anything more than that should be consolidated. Usually, if you have
more than two arguments then your function is trying to do too much. In cases
where it's not, most of the time a higher-level object will suffice as an
argument.

Since JavaScript allows you to make objects on the fly, without a lot of class
boilerplate, you can use an object if you are finding yourself needing a
lot of arguments.

To make it obvious what properties the function expects, you can use the ES2015/ES6
destructuring syntax. This has a few advantages:

1. When someone looks at the function signature, it's immediately clear what
properties are being used.
2. Destructuring also clones the specified primitive values of the argument
object passed into the function. This can help prevent side effects. Note:
objects and arrays that are destructured from the argument object are NOT
cloned.
3. Linters can warn you about unused properties, which would be impossible
without destructuring.

**Bad:**
```javascript
function createMenu(title, body, buttonText, cancellable) {
  // ...
}
```

**Good:**
```javascript
function createMenu({ title, body, buttonText, cancellable }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
})
```
**[⬆ back to top](#table-of-contents)**


### Functions should do one thing
This is by far the most important rule in software engineering and especially
when programming using the Functional Programming approach. When functions
do more than one thing, they are harder to compose, test, and reason about.
When you can isolate a function to just one action, they can be refactored
easily and your code will read much cleaner.

**Bad:**
```javascript
function emailClients(clients) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client)
    if (clientRecord.isActive()) {
      email(client)
    }
  })
}
```

**Good:**
```javascript
function emailClients(clients) {
  clients
    .filter(isClientActive)
    .forEach(email)
}

function isClientActive(client) {
  const clientRecord = database.lookup(client)
  return clientRecord.isActive()
}
```

In general, you should be alerted if you find yourself using "forEach"
that is long or pushing to an array:

**Bad:**
```javascript
const REG_EXES = [
  //...
]

function tokenize(code) {
  const lines = code.split('\n')
  const tokens = []
  lines.forEach((line) => {
    REG_EXES.forEach((regEx) => {
      const match = regEx.exec(line)
      if(match){
        tokens.push(match[1])
      }
    })
  })
  return tokens
}
```

**Good:**
```javascript
const REG_EXES = [
  //...
]

function tokenizeLine(line){
  return REG_EXES
    .map(regEx => regEx.match(line))
    .filter(match => match)
    .map(match => match[1])
}

function tokenize(code, regExes) {
  const lines = code.split('\n')
  const tokensInLines = lines.map(tokenizeLine)
  return _.flatten(tokensInLines)
}
```
**[⬆ back to top](#table-of-contents)**

### Function names should say what they do

**Bad:**
```javascript
function addToDate(date, month) {
  // ...
}

const date = new Date()

// It's hard to to tell from the function name what is added
// and we have to go to the function to understand this.
addToDate(date, 2)
```

**Good:**
```javascript
function addMonthsToDate(month, date) {
  // ...
}

const date = new Date()
addMonthsToDate(2, date)
```
**[⬆ back to top](#table-of-contents)**

### Functions should only be one level of abstraction
When you have more than one level of abstraction your function is usually
doing too much. Splitting up functions leads to reusability and easier
testing.

**Bad:**
```javascript
function parseBetterJSAlternative(code) {
  const REGEXES = [
    // ...
  ]

  const statements = code.split(' ')
  const tokens = []
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    })
  })

  const ast = []
  tokens.forEach((token) => {
    // lex...
  })

  ast.forEach((node) => {
    // parse...
  })
}
```

**Good:**
```javascript
function tokenize(code) {
  const REGEXES = [
    // ...
  ]

  const statements = code.split(' ')
  const tokens = []
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ )
    })
  })

  return tokens
}

function lexer(tokens) {
  const ast = []
  tokens.forEach((token) => {
    ast.push( /* ... */ )
  })

  return ast
}

function parseBetterJSAlternative(code) {
  const tokens = tokenize(code)
  const ast = lexer(tokens)
  ast.forEach((node) => {
    // parse...
  })
}
```
**[⬆ back to top](#table-of-contents)**

### Remove duplicate code
Do your absolute best to avoid duplicate code. Duplicate code is bad because it
means that there's more than one place to alter something if you need to change
some logic.

Oftentimes you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing
duplicate code means creating an abstraction that can handle this set of
different things with just one function/module/class.

**Bad:**
```javascript
function showDeveloperList(developers) {
  developers.forEach((developer) => {
    const expectedSalary = calculateExpectedSalary(developer)
    const experience = getExperience(developer)
    const githubLink = getGithubLink(developer)
    const data = {
      expectedSalary,
      experience,
      githubLink
    }

    render(data)
  })
}

function showManagerList(managers) {
  managers.forEach((manager) => {
    const expectedSalary = calculateExpectedSalary(manager)
    const experience = getExperience(manager)
    const portfolio = getMBAProjects(manager)
    const data = {
      expectedSalary,
      experience,
      portfolio
    }

    render(data)
  })
}
```

**Good:**
```javascript
function getPortofolio(employee){
  return employee.type === 'manager' ?
    getMBAProjects(employee) :
    getGithubLink(employee)
}

function showEmployeeList(employees) {
  employees.forEach((employee) => {
    const expectedSalary = calculateExpectedSalary(employee)
    const experience = getExperience(employee)
    const portfolio = getPortofolio(employee)

    const data = {
      expectedSalary,
      experience,
      portfolio
    }

    render(data)
  })
}
```

**Better:**
```javascript
function getPortofolio(employee){
  return employee.type === 'manager' ?
    getMBAProjects(employee) :
    getGithubLink(employee)
}

function showEmployeeList(employees) {
  employees
    .map(employee => ({
      expectedSalary: calculateExpectedSalary(employee),
      experience: getExperience(employee),
      portfolio: getPortofolio(employee)
    }))
    .forEach(render)
}
```
**[⬆ back to top](#table-of-contents)**

### Set default objects with Object.assign or even better _.defaults

**Bad:**
```javascript
const menuConfig = {
  title: null,
  body: 'Bar',
  buttonText: null,
  cancellable: true
}

function createMenu(config) {
  config.title = config.title || 'Foo'
  config.body = config.body || 'Bar'
  config.buttonText = config.buttonText || 'Baz'
  config.cancellable = config.cancellable === undefined ? config.cancellable : true
}

createMenu(menuConfig)
```

**Good:**
```javascript
const menuConfig = {
  title: 'Order',
  // User did not include 'body' key
  buttonText: 'Send',
  cancellable: true
}

function createMenu(config) {
  config = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config)

  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig)
```

**Better**
```javascript
const menuConfig = {
  title: 'Order',
  // User did not include 'body' key
  buttonText: 'Send',
  cancellable: true
}

function createMenu(config) {
  return _.defaults(config, {
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  })

  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig)
```
**[⬆ back to top](#table-of-contents)**


### Don't use flags as function parameters
Flags tell your user that this function does more than one thing.
Functions should do one thing. Split out your functions if they
are following different code paths based on a boolean.

**Bad:**
```javascript
function createFile(name, temp) {
  if (temp) {
    fs.create(`./temp/${name}`)
  } else {
    fs.create(name)
  }
}
```

**Good:**
```javascript
function createFile(name) {
  fs.create(name)
}

function createTempFile(name) {
  createFile(`./temp/${name}`)
}
```
**[⬆ back to top](#table-of-contents)**

### Don't write to global functions
Polluting globals is a bad practice in JavaScript because you could clash with another
library and the user of your API would be none-the-wiser until they get an
exception in production. Let's think about an example: what if you wanted to
extend JavaScript's native Array method to have a `diff` method that could
show the difference between two arrays? You could write your new function
to the `Array.prototype`, but it could clash with another library that tried
to do the same thing. What if that other library was just using `diff` to find
the difference between the first and last elements of an array? This is why it
would be much better to write a utility or find a library to does what you need.

**Bad:**
```javascript
Array.prototype.diff = function diff(comparisonArray) {
  const hash = new Set(comparisonArray)
  return this.filter(elem => !hash.has(elem))
}
[1, 2].diff([1, 4])
```

**Good:**
```javascript
import {arrDifference} from 'utils'
arrDifference([1, 2], [1, 4])
```

**Better:**
```javascript
_.difference([1, 2], [1, 4])
```
**[⬆ back to top](#table-of-contents)**

### Encapsulate conditionals

**Bad:**
```javascript
if (fsm.state === 'fetching' && isEmpty(listNode)) {
  // ...
}
```

**Good:**
```javascript
function shouldShowSpinner(fsm, listNode) {
  return fsm.state === 'fetching' && isEmpty(listNode)
}

if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```
**[⬆ back to top](#table-of-contents)**

### Avoid negative conditionals

**Bad:**
```javascript
function isDOMNodeNotPresent(node) {
  // ...
}

if (!isDOMNodeNotPresent(node)) {
  // ...
}
```

**Good:**
```javascript
function isDOMNodePresent(node) {
  // ...
}

if (isDOMNodePresent(node)) {
  // ...
}
```
**[⬆ back to top](#table-of-contents)**

### Don't over-optimize
Modern browsers do a lot of optimization under-the-hood at runtime. A lot of
times, if you are optimizing then you are just wasting your time. [There are good
resources](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)
for seeing where optimization is lacking. Target those in the meantime, until
they are fixed if they can be.

**Bad:**
```javascript

// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**Good:**
```javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```
**[⬆ back to top](#table-of-contents)**

### Remove dead code
Dead code is just as bad as duplicate code. There's no reason to keep it in
your codebase. If it's not being called, get rid of it! It will still be safe
in your version history if you still need it.

**Bad:**
```javascript
function oldRequestModule(url) {
  // ...
}

function newRequestModule(url) {
  // ...
}

const req = newRequestModule
inventoryTracker('apples', req, 'www.inventory-awesome.io')

```

**Good:**
```javascript
function requestModule(url) {
  // ...
}

requestModule.inventoryTracker('apples', 'www.inventory-awesome.io')
```
**[⬆ back to top](#table-of-contents)**

## **Objects and Data Structures**
### Use getters and setters
Using getters and setters to access data on objects could be better than simply
looking for a property on an object. "Why?" you might ask. Well, here's an
unorganized list of reasons why:

* When you want to do more beyond getting an object property, you don't have
to look up and change every accessor in your codebase.
* Makes adding validation simple when doing a `set`.
* Encapsulates the internal representation.
* Easy to add logging and error handling when getting and setting.
* You can lazy load your object's properties, let's say getting it from a
server.


**Bad:**
```javascript
function makeBankAccount() {
  // ...

  return {
    balance: 0,
    // ...
  }
}

const account = makeBankAccount()
account.balance = 100
```

**Good:**
```javascript
function makeBankAccount() {
  // this one is private
  let balance = 0

  // a "getter", made public via the returned object below
  function getBalance() {
    return balance
  }

  // a "setter", made public via the returned object below
  function setBalance(amount) {
    // ... validate before updating the balance
    balance = amount
  }

  return {
    // ...
    getBalance,
    setBalance,
  }
}

const account = makeBankAccount()
account.setBalance(100)
```
**[⬆ back to top](#table-of-contents)**


### Make objects have private members
This can be accomplished through closures (for ES5 and below).

**Bad:**
```javascript

const Employee = function(name) {
  this.name = name
}

Employee.prototype.getName = function getName() {
  return this.name
}

const employee = new Employee('John Doe')
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
delete employee.name
console.log(`Employee name: ${employee.getName()}`) // Employee name: undefined
```

**Good:**
```javascript
function makeEmployee(name) {
  return {
    getName() {
      return name
    }
  }
}

const employee = makeEmployee('John Doe')
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
delete employee.name
console.log(`Employee name: ${employee.getName()}`) // Employee name: John Doe
```
**[⬆ back to top](#table-of-contents)**


## **Classes**
### Single Responsibility Principle (SRP)
//TODO: is this too OOP for us?

As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimizing the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify a piece of it,
it can be difficult to understand how that will affect other dependent modules in
your codebase.

**Bad:**
```javascript
class UserSettings {
  constructor(user) {
    this.user = user
  }

  changeSettings(settings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
```

**Good:**
```javascript
class UserAuth {
  constructor(user) {
    this.user = user
  }

  verifyCredentials() {
    // ...
  }
}


class UserSettings {
  constructor(user) {
    this.user = user
    this.auth = new UserAuth(user)
  }

  changeSettings(settings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```
**[⬆ back to top](#table-of-contents)**

### Open/Closed Principle (OCP)
//TODO: is this too OOP for us?

As stated by Bertrand Meyer, "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

**Bad:**
```javascript
class AjaxAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'ajaxAdapter'
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'nodeAdapter'
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter
  }

  fetch(url) {
    if (this.adapter.name === 'ajaxAdapter') {
      return makeAjaxCall(url).then((response) => {
        // transform response and return
      })
    } else if (this.adapter.name === 'httpNodeAdapter') {
      return makeHttpCall(url).then((response) => {
        // transform response and return
      })
    }
  }
}

function makeAjaxCall(url) {
  // request and return promise
}

function makeHttpCall(url) {
  // request and return promise
}
```

**Good:**
```javascript
class AjaxAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'ajaxAdapter'
  }

  request(url) {
    // request and return promise
  }
}

class NodeAdapter extends Adapter {
  constructor() {
    super()
    this.name = 'nodeAdapter'
  }

  request(url) {
    // request and return promise
  }
}

class HttpRequester {
  constructor(adapter) {
    this.adapter = adapter
  }

  fetch(url) {
    return this.adapter.request(url).then((response) => {
      // transform response and return
    })
  }
}
```
**[⬆ back to top](#table-of-contents)**

### Interface Segregation Principle (ISP)
//TODO: is this too OOP for us?

JavaScript doesn't have interfaces so this principle doesn't apply as strictly
as others. However, it's important and relevant even with JavaScript's lack of
type system.

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use." Interfaces are implicit contracts in JavaScript because of
duck typing.

A good example to look at that demonstrates this principle in JavaScript is for
classes that require large settings objects. Not requiring clients to setup
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a "fat interface".

**Bad:**
```javascript
class DOMTraverser {
  constructor(settings) {
    this.settings = settings
    this.setup()
  }

  setup() {
    this.rootNode = this.settings.rootNode
    this.animationModule.setup()
  }

  traverse() {
    // ...
  }
}

const $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  animationModule() {} // Most of the time, we won't need to animate when traversing.
  // ...
})

```

**Good:**
```javascript
class DOMTraverser {
  constructor(settings) {
    this.settings = settings
    this.options = settings.options
    this.setup()
  }

  setup() {
    this.rootNode = this.settings.rootNode
    this.setupOptions()
  }

  setupOptions() {
    if (this.options.animationModule) {
      // ...
    }
  }

  traverse() {
    // ...
  }
}

const $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  options: {
    animationModule() {}
  }
})
```
**[⬆ back to top](#table-of-contents)**

### Dependency Inversion Principle (DIP)
This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

This can be hard to understand at first, but if you've worked with Angular.js,
you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

As stated previously, JavaScript doesn't have interfaces so the abstractions
that are depended upon are implicit contracts. That is to say, the methods
and properties that an object/class exposes to another object/class. In the
example below, the implicit contract is that any Request module for an
`InventoryTracker` will have a `requestItems` method.

**Bad:**
```javascript
class InventoryRequester {
  constructor() {
    this.REQ_METHODS = ['HTTP']
  }

  requestItem(item) {
    // ...
  }
}

class InventoryTracker {
  constructor(items) {
    this.items = items

    // BAD: We have created a dependency on a specific request implementation.
    // We should just have requestItems depend on a request method: `request`
    this.requester = new InventoryRequester()
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item)
    })
  }
}

const inventoryTracker = new InventoryTracker(['apples', 'bananas'])
inventoryTracker.requestItems()
```

**Good:**
```javascript
class InventoryTracker {
  constructor(items, requester) {
    this.items = items
    this.requester = requester
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item)
    })
  }
}

class InventoryRequesterV1 {
  constructor() {
    this.REQ_METHODS = ['HTTP']
  }

  requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 {
  constructor() {
    this.REQ_METHODS = ['WS']
  }

  requestItem(item) {
    // ...
  }
}

// By constructing our dependencies externally and injecting them, we can easily
// substitute our request module for a fancy new one that uses WebSockets.
const inventoryTracker = new InventoryTracker(['apples', 'bananas'], new InventoryRequesterV2())
inventoryTracker.requestItems()
```
**[⬆ back to top](#table-of-contents)**

### Prefer ES2015/ES6 classes over ES5 plain functions
It's very difficult to get readable class inheritance, construction, and method
definitions for classical ES5 classes. If you need inheritance (and be aware
that you might not), then prefer ES2015/ES6 classes. However, prefer small functions over
classes until you find yourself needing larger and more complex objects.

**Bad:**
```javascript
const Animal = function(age) {
  if (!(this instanceof Animal)) {
    throw new Error('Instantiate Animal with `new`')
  }

  this.age = age
}

Animal.prototype.move = function move() {}

const Mammal = function(age, furColor) {
  if (!(this instanceof Mammal)) {
    throw new Error('Instantiate Mammal with `new`')
  }

  Animal.call(this, age)
  this.furColor = furColor
}

Mammal.prototype = Object.create(Animal.prototype)
Mammal.prototype.constructor = Mammal
Mammal.prototype.liveBirth = function liveBirth() {}

const Human = function(age, furColor, languageSpoken) {
  if (!(this instanceof Human)) {
    throw new Error('Instantiate Human with `new`')
  }

  Mammal.call(this, age, furColor)
  this.languageSpoken = languageSpoken
}

Human.prototype = Object.create(Mammal.prototype)
Human.prototype.constructor = Human
Human.prototype.speak = function speak() {}
```

**Good:**
```javascript
class Animal {
  constructor(age) {
    this.age = age
  }

  move() { /* ... */ }
}

class Mammal extends Animal {
  constructor(age, furColor) {
    super(age)
    this.furColor = furColor
  }

  liveBirth() { /* ... */ }
}

class Human extends Mammal {
  constructor(age, furColor, languageSpoken) {
    super(age, furColor)
    this.languageSpoken = languageSpoken
  }

  speak() { /* ... */ }
}
```
**[⬆ back to top](#table-of-contents)**


### Use method chaining
This pattern is very useful in JavaScript and you see it in many libraries such
as jQuery and Lodash. It allows your code to be expressive, and less verbose.
For that reason, I say, use method chaining and take a look at how clean your code
will be. In your class functions, simply return `this` at the end of every function,
and you can chain further class methods onto it.

**Bad:**
```javascript
class Car {
  constructor() {
    this.make = 'Honda'
    this.model = 'Accord'
    this.color = 'white'
  }

  setMake(make) {
    this.make = make
  }

  setModel(model) {
    this.model = model
  }

  setColor(color) {
    this.color = color
  }

  save() {
    console.log(this.make, this.model, this.color)
  }
}

const car = new Car()
car.setColor('pink')
car.setMake('Ford')
car.setModel('F-150')
car.save()
```

**Good:**
```javascript
class Car {
  constructor() {
    this.make = 'Honda'
    this.model = 'Accord'
    this.color = 'white'
  }

  setMake(make) {
    this.make = make
    // NOTE: Returning this for chaining
    return this
  }

  setModel(model) {
    this.model = model
    // NOTE: Returning this for chaining
    return this
  }

  setColor(color) {
    this.color = color
    // NOTE: Returning this for chaining
    return this
  }

  save() {
    console.log(this.make, this.model, this.color)
    // NOTE: Returning this for chaining
    return this
  }
}

const car = new Car()
  .setColor('pink')
  .setMake('Ford')
  .setModel('F-150')
  .save()
```
**[⬆ back to top](#table-of-contents)**

### Prefer composition over inheritance
As stated famously in [*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four,
you should prefer composition over inheritance where you can. There are lots of
good reasons to use inheritance and lots of good reasons to use composition.
The main point for this maxim is that if your mind instinctively goes for
inheritance, try to think if composition could model your problem better. In some
cases it can.

You might be wondering then, "when should I use inheritance?" It
depends on your problem at hand, but this is a decent list of when inheritance
makes more sense than composition:

1. Your inheritance represents an "is-a" relationship and not a "has-a"
relationship (Human->Animal vs. User->UserDetails).
2. You can reuse code from the base classes (Humans can move like all animals).
3. You want to make global changes to derived classes by changing a base class.
(Change the caloric expenditure of all animals when they move).

**Bad:**
```javascript
class Employee {
  constructor(name, email) {
    this.name = name
    this.email = email
  }

  // ...
}

// Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee
class EmployeeTaxData extends Employee {
  constructor(ssn, salary) {
    super()
    this.ssn = ssn
    this.salary = salary
  }

  // ...
}
```

**Good:**
```javascript
class EmployeeTaxData {
  constructor(ssn, salary) {
    this.ssn = ssn
    this.salary = salary
  }

  // ...
}

class Employee {
  constructor(name, email) {
    this.name = name
    this.email = email
  }

  setTaxData(ssn, salary) {
    this.taxData = new EmployeeTaxData(ssn, salary)
  }
  // ...
}
```
**[⬆ back to top](#table-of-contents)**

## **Testing**
Testing is more important than shipping. If you have no tests or an
inadequate amount, then every time you ship code you won't be sure that you
didn't break anything. Deciding on what constitutes an adequate amount is up
to your team, but having 100% coverage (all statements and branches) is how
you achieve very high confidence and developer peace of mind. This means that
in addition to having a great testing framework, you also need to use a
[good coverage tool](http://gotwarlost.github.io/istanbul/).

There's no excuse to not write tests. There's [plenty of good JS test frameworks]
(http://jstherightway.org/#testing-tools), so find one that your team prefers.
When you find one that works for your team, then aim to always write tests
for every new feature/module you introduce. If your preferred method is
Test Driven Development (TDD), that is great, but the main point is to just
make sure you are reaching your coverage goals before launching any feature,
or refactoring an existing one.

### Single concept per test

**Bad:**
```javascript
const assert = require('assert')

describe('MakeMomentJSGreatAgain', () => {
  it('handles date boundaries', () => {
    let date

    date = new MakeMomentJSGreatAgain('1/1/2015')
    date.addDays(30)
    assert.equal('1/31/2015', date)

    date = new MakeMomentJSGreatAgain('2/1/2016')
    date.addDays(28)
    assert.equal('02/29/2016', date)

    date = new MakeMomentJSGreatAgain('2/1/2015')
    date.addDays(28)
    assert.equal('03/01/2015', date)
  })
})
```

**Good:**
```javascript
const assert = require('assert')

describe('MakeMomentJSGreatAgain', () => {
  it('handles 30-day months', () => {
    const date = new MakeMomentJSGreatAgain('1/1/2015')
    date.addDays(30)
    assert.equal('1/31/2015', date)
  })

  it('handles leap year', () => {
    const date = new MakeMomentJSGreatAgain('2/1/2016')
    date.addDays(28)
    assert.equal('02/29/2016', date)
  })

  it('handles non-leap year', () => {
    const date = new MakeMomentJSGreatAgain('2/1/2015')
    date.addDays(28)
    assert.equal('03/01/2015', date)
  })
})
```
**[⬆ back to top](#table-of-contents)**

## **Concurrency**
### Use Promises, not callbacks
Callbacks aren't clean, and they cause excessive amounts of nesting. With ES2015/ES6,
Promises are a built-in global type. Use them!

**Bad:**
```javascript
require('request').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', (requestErr, response) => {
  if (requestErr) {
    console.error(requestErr)
  } else {
    require('fs').writeFile('article.html', response.body, (writeErr) => {
      if (writeErr) {
        console.error(writeErr)
      } else {
        console.log('File written')
      }
    })
  }
})

```

**Good:**
```javascript
require('request-promise').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then((response) => {
    return require('fs-promise').writeFile('article.html', response)
  })
  .then(() => {
    console.log('File written')
  })
  .catch((err) => {
    console.error(err)
  })

```
**[⬆ back to top](#table-of-contents)**

### Async/Await are even cleaner than Promises
Promises are a very clean alternative to callbacks, but ES2017/ES8 brings async and await
which offer an even cleaner solution. All you need is a function that is prefixed
in an `async` keyword, and then you can write your logic imperatively without
a `then` chain of functions. Use this if you can take advantage of ES2017/ES8 features
today!

**Bad:**
```javascript
require('request-promise').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then((response) => {
    return require('fs-promise').writeFile('article.html', response)
  })
  .then(() => {
    console.log('File written')
  })
  .catch((err) => {
    console.error(err)
  })

```

**Good:**
```javascript
async function getCleanCodeArticle() {
  try {
    const response = await require('request-promise').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
    await require('fs-promise').writeFile('article.html', response)
    console.log('File written')
  } catch(err) {
    console.error(err)
  }
}
```
**[⬆ back to top](#table-of-contents)**


## **Error Handling**
Thrown errors are a good thing! They mean the runtime has successfully
identified when something in your program has gone wrong and it's letting
you know by stopping function execution on the current stack, killing the
process (in Node), and notifying you in the console with a stack trace.

### Don't ignore caught errors
Doing nothing with a caught error doesn't give you the ability to ever fix
or react to said error. Logging the error to the console (`console.log`)
isn't much better as often times it can get lost in a sea of things printed
to the console. If you wrap any bit of code in a `try/catch` it means you
think an error may occur there and therefore you should have a plan,
or create a code path, for when it occurs.

**Bad:**
```javascript
try {
  functionThatMightThrow()
} catch (error) {
  console.log(error)
}
```

**Good:**
```javascript
try {
  functionThatMightThrow()
} catch (error) {
  // One option (more noisy than console.log):
  console.error(error)
  // Another option:
  notifyUserOfError(error)
  // Another option:
  reportErrorToService(error)
  // OR do all three!
}
```

### Don't ignore rejected promises
For the same reason you shouldn't ignore caught errors
from `try/catch`.

**Bad:**
```javascript
getdata()
  .then((data) => {
    functionThatMightThrow(data)
  })
  .catch((error) => {
    console.log(error)
  })
```

**Good:**
```javascript
getdata()
  .then((data) => {
    functionThatMightThrow(data)
  })
  .catch((error) => {
    // One option (more noisy than console.log):
    console.error(error)
    // Another option:
    notifyUserOfError(error)
    // Another option:
    reportErrorToService(error)
    // OR do all three!
  })
```

**[⬆ back to top](#table-of-contents)**

## **Comments**
### Only comment things that have business logic complexity.
Comments are an apology, not a requirement. Good code *mostly* documents itself.

**Bad:**
```javascript
function hashIt(data) {
  // The hash
  let hash = 0

  // Length of string
  const length = data.length

  // Loop through every character in data
  for (let i = 0; i < length; i++) {
    // Get character code.
    const char = data.charCodeAt(i)
    // Make the hash
    hash = ((hash << 5) - hash) + char
    // Convert to 32-bit integer
    hash &= hash
  }
}
```

**Good:**
```javascript

function hashIt(data) {
  let hash = 0
  const length = data.length

  for (let i = 0; i < length; i++) {
    const char = data.charCodeAt(i)
    hash = ((hash << 5) - hash) + char

    // Convert to 32-bit integer
    hash &= hash
  }
}

```
**[⬆ back to top](#table-of-contents)**

### Don't leave commented out code in your codebase
Version control exists for a reason. Leave old code in your history.

**Bad:**
```javascript
doStuff()
// doOtherStuff()
// doSomeMoreStuff()
// doSoMuchStuff()
```

**Good:**
```javascript
doStuff()
```
**[⬆ back to top](#table-of-contents)**

### Don't have journal comments
Remember, use version control! There's no need for dead code, commented code,
and especially journal comments. Use `git log` to get history!

**Bad:**
```javascript
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
function combine(a, b) {
  return a + b
}
```

**Good:**
```javascript
function combine(a, b) {
  return a + b
}
```
**[⬆ back to top](#table-of-contents)**

### Avoid positional markers
They usually just add noise. Let the functions and variable names along with the
proper indentation and formatting give the visual structure to your code.

**Bad:**
```javascript
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
$scope.model = {
  menu: 'foo',
  nav: 'bar'
}

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
const actions = function() {
  // ...
}
```

**Good:**
```javascript
$scope.model = {
  menu: 'foo',
  nav: 'bar'
}

const actions = function() {
  // ...
}
```
**[⬆ back to top](#table-of-contents)**