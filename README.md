# code-review-guide

## Table of Contents

1. [Introduction](#introduction)
2. [Why Review Code?](#why-review-code)
3. [Basics](#basics)
4. [Readability](#readability)
5. [Side Effects](#side-effects)
6. [Limits](#limits)
7. [Security](#security)
8. [Performance](#performance)
9. [Testing](#testing)
10. [Miscellaneous](#miscellaneous)

## Introduction

Code reviews often inspire dread in both the reviewer and reviewee. Having your
code analyzed can feel invasive and uncomfortable. At the same time, reviewing
other developers' code can be feel like a painful, ambiguous exercise,

This document aims to provide some solid foundational rules for how to review
the code that we write. All examples are written in JavaScript, but the advice
is be applicable to any project in any language. This is by no means an
exhaustive list, but hopefully this will help us catch as many bugs as possible
long before our customers ever see our apps and features.

## Why Review Code?

Code reviews are a necessary part of the software engineering process because
developers are human. We simply can't catch every problem in every piece of
code we write. Code reviews are the primary way we ensure that at least a
second pair of eyes has been over every line of code we put into production -
to act as a sanity check, catch brain farts, and to prevent technical debt
down the line.

Having others review our work ensures that we deliver the best product to our
customers, with the fewest errors possible.

It's important to note that there is no one-size-fits-all code review process.
The important thing is to perform code reviews as regularly as possible.

## Basics

### Code reviews should be as automated as possible

To make code reviews as productive as possible - and minimize friction amongst
one another - it's best to avoid discussing subjective details such as the use
`getAllItems` or `allItemsGetter`. Code style should be decided upon in advance
and enforced by automated linting and formatting tools.

### Code reviews should avoid API discussion

These discussions should happen before the code is ever written. If at all
possible, don't re-arrange the floor plan after the foundation has already been
laid!

### Code reviews should be kind

It's scary to have your code reviewed, and it can bring about feelings of
insecurity in even the most experienced developer. Be positive in your language
(or, at least try to avoid negative language) and keep your teammates
comfortable and secure in their work!

## Readability

### Typos should be corrected

Avoid nitpicking as much as you can and save it for your linter, compiler, and
formatter. When you can't, such as in the case of typos, leave a kind comment
suggesting a fix. It's often the little things that make a big difference!

### Variable and function names should be clear

Naming is one of the hardest problems in computer science. We've all given names
to variables, functions, and files that are confusing. Help your colleagues out
by suggesting a clearer name, if the one you're reading doesn't make sense.

```javascript
// This function could be better named as namesToUpperCase
function u(names) {
  // ...
}
```

### Functions should be short

Functions should do one thing! Long functions usually mean that they are doing
too much.

A good rule of thumb is that if you have to use the word "and" in describing
what a function does, it's time to split out the function into multiple
different functions instead.

```javascript
// This is both emailing clients and deciding which are active. Should be
// 2 different functions.
function emailClients(clients) {
  clients.forEach(client => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

### Files should be cohesive, and ideally short

Just like functions, a file should be about one thing. A file represents a
module and a module should do one thing for your codebase.

For example, if your module is called `fakeNameGenerator` it should just be
responsible for creating fake names like "Angela Cadwell". If the
`fakeNameGenerator` also includes a bunch of utility functions for querying a
database of names, that should be in a separate module.

There's no rule for how long a file should be, but if it's long like below **and**
includes functions that don't relate to one another, then it should probably
be split apart.

```javascript
// Line 1
import _ from 'lodash';
function generateFakeNames() {
  // ..
}

// Line 1128
function queryRemoteDatabase() {
  // ...
}
```

### Exported functions should be documented

If your function is intended to be used by other libraries or modules, it helps
to add documentation so other developers know what it does.

```javascript
// This needs documentation. What is this function for? How is it used?
export function networkMonitor(graph, duration, failureCallback) {
  // ...
}
```

### Complex code should be commented

If you have named things well and the logic is still confusing, then it's time
for a comment.

```javascript
function leftPad(str, len, ch) {
  str = str + '';
  len = len - str.length;

  while (true) {
    // This needs a comment, why a bitwise and here?
    if (len & 1) pad += ch;
    // This needs a comment, why a bit shift here?
    len >>= 1;
    if (len) ch += ch;
    else break;
  }

  return pad + str;
}
```

## Side Effects

### Functions should be as pure as possible

A pure function is a function that will always return the same value given the
same parameters, and **produces no side effects**. A dead giveaway that a
function is impure is if it makes sense to call it without using its return
value.

Obviously, it's impossible to right useful software if *every* function is
pure, but by itself, this is the single most important thing you can do to
reduce the number of bugs in your code.

```javascript
// Global variable is referenced by the following function.
// If we had another function that used this name, now it'd be an array and it
// could break it. Instead it's better to pass in a name parameter
let name = 'Adil Asif';

function splitIntoFirstAndLastName() {
  name = name.split(' ');
}

splitIntoFirstAndLastName();
```

### I/O functions should have failure cases handled

Any function that performs I/O should handle cases when something goes wrong.

```javascript
function getIngredientsFromFile() {
  const onFulfilled = buffer => {
    let lines = buffer.split('\n');
    return lines.forEach(line => <Ingredient ingredient={line} />);
  };

  // What about when this rejected because of an error? What do we return?
  return readFile('./ingredients.txt').then(onFulfilled);
}
```

## Limits

### Null cases should be handled

If you have a list component for example, all is well and good if you display a
beautiful table that shows all its data. But what happens when no data comes
back because of a failure in the API or database? What do you show in the null
case? Your code should be resilient to every case that can *realistically*
occur. If there's something bad that can happen in your code, eventually, it
will happen.

```javascript
class InventoryList {
  constructor(data) {
    this.data = data;
  }

  render() {
    return (
      <table>
        <tbody>
          <tr>
            <th>ID</th>
            <th>Product</th>
          </tr>
          // We should show something for the null case here if there's
          // nothing in the data inventory
          {Object.keys(this.data.inventory).map(itemId => (
            <tr key={i}>
              <td>{itemId}</td>

              <td>{this.state.inventory[itemId].product}</td>
            </tr>
          ))}
        </tbody>
      </table>
    );
  }
}
```

### Large cases should be handled

In the list above, what would happen if 20,000 items were returned in
`data.inventory`? In that case, we need some form of pagination or infinite-
scrolling. Be sure to always assess the potential edge cases in terms of
volume, particularly when it comes to UI programming!

### Singular cases should be handled

```javascript
class MoneyDislay {
  constructor(amount) {
    this.amount = amount;
  }

  render() {
    // What happens if the user has 1 dollar? You can't say plural "dollars"
    return (
      <div className="fancy-class">
        You have {this.amount} dollars in your account
      </div>
    );
  }
}
```

### User input should be limited

Users can potentially input an unlimited amount of data to send to you. It's
important to set limits if a function takes any kind of user data in.

```javascript
router.route('/message').post((req, res) => {
  const message = req.body.content;

  // What happens if the message is many megabytes of data? Do we want to store
  // that in the database? We should set limits on the size.
  db.save(message);
});
```

### Functions should handle unexpected user input

Users will always surprise you with the data they give you. Don't expect that
you will always get the right type of data or even any data in a request from a
user. [And don't rely on client-side validation alone!](https://i.redd.it/ldvkdq3cc02y.jpg)

```javascript
router.route('/transfer-money').post((req, res) => {
  const amount = req.body.amount;
  const from = user.id;
  const to = req.body.to;

  // What happens if we got a string instead of a number as our amount? This
  // function would fail
  transferMoney(from, to, amount);
});
```

## Security

Data security is the most important aspect of an application. There are numerous
different types of security exploits that can plague an app, depending on the
particular language and runtime environment. Below is a very small and very
incomplete list of common security problems. Don't rely on this alone! Automate
as much security review as you can on every commit, and perform routine security
audits.

### XSS should not be possible

Cross-site scripting (XSS), the largest single vector for security attacks
on a web application. It occurs when you take user input and include it in your
page without first properly sanitizing it. This can cause your site to execute
source code from remote pages.

```javascript
function getBadges() {
  let badge = document.getElementsByClassName('badge');
  let nameQueryParam = getQueryParams('name');

  /**
   * What if nameQueryParam was `<script>sendCookie(document.cookie)</script>`?
   * If that was the query param, a malicious user could lure a user to click a
   * link with that as the `name` query param, and have the user unknowingly
   * send their data to a bad actor.
   */
  badge.children[0].innerHTML = nameQueryParam;
}
```

### Personally Identifiable Information (PII) should not leak

We bear an enormous weight of responsibility every time we take in user data.
If we leak data in URLs, in analytics tracking to third parties, or even expose
data to employees that shouldn't have access, we greatly hurt our users and
our business. Be careful with other people's lives!

```javascript
router.route('/bank-user-info').get((req, res) => {
  const name = user.name;
  const id = user.id;
  const socialSecurityNumber = user.ssn;

  // There's no reason to send a socialSecurityNumber back in a query parameter
  // This would be exposed in the URL and potentially to any middleman on the
  // network watching internet traffic
  res.addToQueryParams({
    name,
    id,
    socialSecurityNumber
  });
});
```

## Performance

### Functions should use efficient algorithms and data structures

This is different for every particular case, but use your best judgment to see
if there are any ways to improve the efficiency of a piece of code. Your users
will thank you for the faster speeds!

```javascript
// If mentions was a hash data structure, you wouldn't need to iterate through
// all mentions to find a user. You could simply return the presence of the
// user key in the mentions hash
function isUserMentionedInComments(mentions, user) {
  let mentioned = false;
  mentions.forEach(mention => {
    if (mention.user === user) {
      mentioned = true;
    }
  });

  return mentioned;
}
```

### Important actions should be logged

Logging helps give metrics about performance and insight into user behavior.
Not every action needs to be logged, but decide with your team what makes sense
to keep track of for data analytics. And be sure that no personally identifiable
information is exposed!

```javascript
router.route('/request-ride').post((req, res) => {
  const currentLocation = req.body.currentLocation;
  const destination = req.body.destination;

  requestRide(user, currentLocation, destination).then(result => {
    // We should log before and after this block to get a metric for how long
    // this task took, and potentially even what locations were involved in ride
    // ...
  });
});
```

## Testing

### New code should be tested

Ideally, all new code should include a test, whether it fixes a bug, or is a
new feature. If it's a bugfix, it should have a test proving that the bug is
fixed, and if it's a new feature, then every stateful or functional component
should be unit tested, and there should be an integration test ensuring that
the feature doesn't break the rest of the system.

### Tests should actually test all of what the function does

```javascript
function payEmployeeSalary(employeeId, amount, callback) {
  db.get('EMPLOYEES', employeeId)
    .then(user => {
      return sendMoney(user, amount);
    })
    .then(res => {
      if (callback) {
        callback(res);
      }

      return res;
    });
}

const callback = res => console.log('called', res);
const employee = createFakeEmployee('john jacob jingleheimer schmidt');
const result = payEmployeeSalary(employee.id, 1000, callback);
assert(result.status === enums.SUCCESS);
// What about the callback? That should be tested
```

### Tests should stress edge cases and limits of a function

```javascript
function dateAddDays(dateTime, day) {
  // ...
}

let dateTime = '1/1/2019';
let date1 = dateAddDays(dateTime, 5);

assert(date1 === '1/6/2019');

// What happens if we add negative days?
// What happens if we add fractional days: 1.2, 8.7, etc.
// What happens if we add 1 billion days?
```

## Miscellaneous

> _"Everything can be filed under miscellaneous"_

> George Bernard Shaw

### TODO comments should be tracked

TODO comments are great for letting you and your fellow engineers know that
something needs to be fixed later. Sometimes you simply have ship code and wait
to fix it later. But eventually, you'll have to clean it up! That's why you
should track it - and ideally, give a corresponding ID from your issue tracking
system so you can schedule it and keep track of where the problem is in your
codebase.

### Commit messages should be clear and accurately describe new code

We've all written commit messages like "Changed some crap", "wip",
"ugh, one more to fix this damn bug". These are funny and satisfying, but not
helpful when you're up on a Sunday morning because you pushed code last Friday
night and can't figure out what the bad code was doing when you `git blame` the
commit. Write commit messages that describe the code accurately, and include
a ticket number from your issue tracking system if you have one. That will make
searching through your commit log much easier.

### The code should do what it's supposed to do

This seems obvious, but most reviewers don't have the time or take the time to
manually test every user-facing change. It's important to make sure the
business logic of every change is as per design. It's easy to forget that when
you're just looking for problems in the code!
