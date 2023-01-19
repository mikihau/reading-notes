Unary plus converts strings to numbers
```javascript
+ '42';   // 42
+ '010';  // 10
+ '0x10'; // 16 (strings starts with 0x is hex based)
```

Plusing strings with numbers
```javascript
'3' + 4 + 5;  // "345"
 3 + 4 + '5'; // "75"
```

Unary plus vs parseInt/parseFloat
```javascript
let str = "10.2abc";  // undefined
parseInt(str);        // 10
parseFloat(str);      // 10.2
+ str;                // NaN
```

Prefer Number.isNaN() to global isNaN() if you only want to check for NaN
```javascript
Number.isNaN(undefined);  // false
isNaN(undefined);         // true
```

Returning values of declarations and assignments
```javascript
let variable;     // undefined
let variable = 1; // undefined
variable = 2;     // 2 (assignments return the value it's assigned)
```

== vs ===
```javascript
123 == '123';    // true, == performs type coercion first
123 === '123';   // false
1 == true;       // true, coerces type
```

## Looping
```javascript
// #1 java style
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// #2 (iterable only) for...of
// works for any iterable object -- array, string, Map, Set, arguments, iterator, generator etc
for (let element of [1,2,3]) { // can also use destructuring for the looping variable
  console.log(element);
}

// #3 (object only) for..in
for (let property in {'name': 'john', 'age': 5}) {
  console.log(property);
}

// #4 (array only) forEach
['egg', 'milk', 'butter'].forEach(
  (value, index, array) => {
    console.log(`value: ${value}, index: ${index}, array: ${array}`);
  }
);
```

## [Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
Function with indefinite number of arguments with rest parameters
```javascript
function display(first, ...rest) {
  console.log(first); // 1
  console.log(rest); // [2,3,4]
}
display(1, 2, 3, 4);
```

Destructuring an array
```javascript
const myArray = ["a", "b", "c"];
const [x, y] = myArray      // now x === 'a' && y === 'b'
```

Destructuring an object
```javascript

```

Object property shorthand for properties of the same name
```javascript
const [p, q] = [10, 20];

// this 
const myObj = {
  p: p, 
  q: q 
};

// is equivalent to
const myObj = {p, q};
```

Spreading an array/object/string into arguments with the spread operator
```javascript
// can be used for copy/concates shallowly
obj1 = {'a': 1};
obj2 = {'b': 2}
{...obj1, ...obj2} // {a: 1, b: 2}
```

Making a custom object
```javascript
function Person(first, last) {
  this.first = first;
  this.last = last;
}
Person.prototype.fullName = function() {          // define a method on the object's prototype chain
  return this.first + ' ' + this.last;            // this refers to the current object inside a function
};
Person.prototype.fullNameReversed = function() {
  return this.last + ', ' + this.first;
};
var s = new Person('Simon', 'Willison');          // 'new' creates an empty object and calls the function with 'this' set to the new object, and returns the object
```

Making a custom object using the 'class' sugar
```javascript
class Person {
  constructor(first, last) {
    this.first = first;
    this.last = last;
  }

  fullName() {
    return this.first + ' ' + this.last;
  }
  
  static getScientificName() {
    return "Homo sapiens";
  }
}
Person.getScientificName();
```

Destructuring an object
```javascript
const person = {
  firstName: "Nick",
  lastName: "Anderson",
  age: 35,
}

// this block
const first = person.firstName;
const age = person.age;
const city = person.city || "Paris";

// is equivalent to
const { firstName: first, age, city = "Paris" } = person;  // {} is the destructuring syntax here, not a block or object

// in function parameters
const joinFirstLastName = ({ firstName, lastName }) => firstName + '-' + lastName;
```

Returning synchronously with an asynchronous function with a Promise
```javascript
// 3 states of a Promise: pending, fulfilled, rejected
const xFetcherPromise = new Promise(
  (resolve, reject) => {
    $.get("X")                         // launch the ajax request
      .done((X) => resolve(X))
      .fail((error) => reject(error));
  }
)

xFetcherPromise
  .then((X) => console.log(X))        // provide the resolve function
  .catch((err) => console.log(err))   // provide the error function
```

Chaining asynchronous calls with async/await
```javascript
// this function
function getGithubUser(username) {
  return fetch(`https://api.github.com/users/${username}`).then(response => response.json()); // fetch returns a promise
}

// is equivalent to
async function getGithubUser(username) { // an async function always returns a promise
  const response = await fetch(`https://api.github.com/users/${username}`); // execution is paused for the await until the Promise returned by fetch is resolved
  return response.json();
}

// usage
getGithubUser('mbeaudru')
  .then(user => console.log(user))
  .catch(err => console.log(err)); // any uncaught exceptions in the async function results in a reject
```

Generating iterables with generators
```javascript
// Generators are functions that can be exited and later re-entered with its context (variable bindings) saved across re-entrances.
function * genB(i) {      // declares a generator function
  yield i + 1;
  yield i + 2;
}

function * genA(i) {
  yield i;
  yield* genB(i);         // calls another generator function
  yield i + 10;
}

var gen = genA(10);

gen.next(); // { value: 10, done: false }
gen.next(); // { value: 11, done: false }
gen.next(); // { value: 12, done: false }
gen.next(); // { value: 20, done: false }
```

---
Reference: 
- [A re-introduction to JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
- [Modern Javascript Cheatsheet](https://github.com/mbeaudru/modern-js-cheatsheet)
