# UI Contributing Guidelines

## How to change FrontEnd coding guidelines

When at a crossroads on the best course of action, please remember that `consistency` is an extremely vital characteristic of any large codebase. Software architecture is a topology. Having consistent code works as a map for every software engineer who needs to find/understand/extend/write logic. Good DX and intuitive code are critical concepts that could be more important than performance, memory usage, or DRY paradigm.

## 1. Updating npm packages

#### (1.1) When updating an npm package, use strict versioning, ie. an exact version number in the package.json.

- The caret ^ means "compatible with version" and will install any minor/patch versions greater than or equal to the version specified.

- The tilde ~ means "approximately equivalent to version" and will install any future patch versions after the one specified.

- Use of the above can lead to inconsistencies in local environments, untrustworthy or buggy new versions of packages being installed, and more issues that are difficult to debug.

- Strict versioning means npm will install only the exact version we specify.

```javascript
// Bad
"del-cli": "^4.0.0",

// Use carefully
"del-cli": "~4.0.0",

// Good
"del-cli": "4.0.0",
```

## 2. Typescript / JavaScript

#### (2.1) Use undefined instead of null

> (2.1.1) We do not use `null` in our project, so please remember to use `undefined` instead. There is no need to use two Null values for variables. `null` also does not work with default function parameters and deconstructing. It can also increase the code if you want to check if a value is missing.

> (2.1.2) We have middlewares that would convert any `null` to `undefined` that is returned from our API

> (2.1.3) Would we ever need a `null` value? It is required by React when we want to return an empty element. Another possible place where we would want it is API and partial updates. But we do not support partial updates (yet), so we opted out from `null` values to take advantage of ES6 features.

#### (2.2) Leave comments for your comments

Leave comments for commented out lines of code indicating WHY it is commented and WHEN it will be
uncommented

- There is a high chance that another developer will have to look at the commented out code in the future and make sense if it still needed or not

```javascript
// Bad

// function update() {
//  test = `${test}-Updated`;
// }

// Good

// Commenting this function since we are not using it anymore in the new code
// Remove it in nearest future if we do not rollback to the old modules implementation
// function update() {
// test = `${test}-Updated`;
// }

// Bad
const params = {
  state,
  locationId,
  date,

  // offset,
  // limit
};

// Good
const params = {
  state,
  locationId,
  date,

  // commenting out offset and limit since we want to return all the rows back
  // keeping this code if we need to go back to pagination. If not remove it in future
  // offset,
  // limit
};
```

#### (2.3) Do not leave useless comments. Use comments to describe complicated logic or describe things that are hard to read in the code.

- Try to write readable code. Developers spend most of their time reading someone else's code rather than writing it. Leaving your code in unreadable state or adding redundant comments can contribute to "developer fatigue". Try to keep things clean.

(2.3.1) Avoid useless comments

```javascript
// Bad

// --- Event Functions --- //
function createEvent() {}

function deleteEvent() {}

// Good

function createEvent() {}

function deleteEvent() {}
```

(2.3.2) Make code readable instead of adding comments explaining what you wrote

```javascript
// Bad

// This is a function to to capitalize the first letter of the string
export function cfl(s) {
  if (typeof s !== "string") {
    return "";
  }
  const [firstLetter = ""] = s;

  return firstLetter.toUpperCase() + s.substr(1);
}

// Good
export function capitalizeFirstLetter(value = "") {
  if (typeof value !== "string" || value === "") {
    return "";
  }
  const [firstLetter = ""] = value;

  return firstLetter.toUpperCase() + value.substr(1);
}
```

#### (2.4) Follow functional programming paradigm. Try to avoid changing variables which are not in the scope. Also be mindful of passed arrays or objects not to modify initially passed references.

```javascript
// Bad
let test = "";

function update() {
  test = `${test}-Updated`;
}

// Good
let test = "";

function update(str) {
  return `${str}-Updated`;
}

test = update(test);

// Bad
function changeMember(x = {}) {
  x.member = "bar"; // this operation will modify original passed argument into changeMember()
}

// Good
function changeMember(x = {}) {
  return { ...x, member: "bar" };
}

// Bad
function changeMember(array = []) {
  array[0] = "bar"; // this operation will modify original passed argument into changeMember()
}

// Good
function changeMember(array = []) {
  return ["bar", ...array];
}
```

#### (2.5) Use functional programming approaches to permutate or retrieve data from arrays and objects

- You can find more data manipulation functions here: https://30secondsofcode.org/

```javascript
// push element to the beginning of the array
const result = [element, ...array];

// push element to the end of the array
const result = [...array, element];

// add element to a specific position in the array
const result = [...array.slice(0, index), element, ...array.slice(index + 1)];

// remove element with a specified index
const result = [...array.slice(0, index - 1), ...array.slice(index + 1)];

// unique values in array
const result = [...new Set(array)];

// calculate sum of all of the integeres in array
const sum = array.reduce((result, element) => {
  return result + element;
}, 0);

// get first element from non-empty array
const [firstElement] = array;

// get the rest of array elements except the first element
const [firstElement, ...rest] = array; // rest is the desired array

// overwrite object properties
const result = { ...object, ...changes, arg1: value1, arg2: value2 };

// get all object properties except of id
const { id, ...rest } = object; // rest is a desired object

// remove object from an array of objects by id
const result = objects.filter((object) => object.id !== id);

// remove all non-empty, non-falsy, non-zero elements from an array
const result = array.filter((e) => e);

// get array of IDs from the array of objects
const result = objects.map((object) => object.id);

// find if at least one of the objects fit the criteria
const result = objects.some((object) => object.arg === value);

// find if all of the objects fit the criteria
const result = objects.every((object) => object.arg === value);

// modify object in array of objects by id
const result = objects.map((object) => {
  if (object.id !== id) {
    return object;
  }

  return { ...object, ...change };
});
```

#### (2.6) Look up existing utils files before adding new helper functions

- Be critical about adding new helper functions instead of re-using or extending existing ones. Only do it when absolutly necessary and ensure any existing UI expected to adhere to the new logic is updated.

- Remember that a lot of helper functions are designed to encapsulate business rules and should be the source of thruth for following required business logic. Creating multiple functions to follow the same set of rules opens a door to inconsistency in functions implementation and maintenance. In addition, it contributes to spaghetti-code and lack of consistency in implementation when developers do not know which function to use for new features.

- Provide detailed and clear description of the function, its purpose and its output
- Write unit tests to ensure that future changes will not disrupt expected behaviour

```javascript
// The one and only function to generate UI label for the range of two datetime values
// Today/Tomorrow/Yesterday always trumps explicit date
// Pre-condition for the example below: today is April 21, 2020

// date(+time) to the same date(+time) range: Today @ 13:00 - 14:00               (start=date(+time), end=date(+time), showStartTime=true, showEndTime=true, delimiter='@'}
// date(+time) to date range:                 Today @ 13:00 - Tomorrow            (start=date(+time), end=date, showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=true, showEndTime=false,delimiter='@'}
// date to date(+time) range:                 Yesterday - Today @ 15:00           (start=date, end=date(+time), showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=false, showEndTime=true,delimiter='@'}
// date to date range:                        Yesterday - Today                   (start=date, end=date, showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=false,showEndTime=false,delimiter='@'}
// date to the same date range:               Yesterday                           (start===end=date, showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start===end=date(+time), showStartTime=false, showEndTime=false, delimiter='@'}

// If required to use explicit dates:
// date(+time) to the same date(+time) range: Apr 25, 2020 @ 13:00 - 14:00        (start=date(+time), end=date(+time), showStartTime=true, showEndTime=true, delimiter='@'}
// date(+time) to date range:                 Apr 25, 2020 @ 13:00 - Apr 26, 2020 (start=date(+time), end=date,showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=true, showEndTime=false,delimiter='@'}
// date to date(+time) range:                 Apr 25, 2020 - Apr 26, 2020 @ 15:00 (start=date, end=date(+time), showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=false, showEndTime=true,delimiter='@'}
// date to date range:                        Apr 25, 2020 - Apr 26, 2020         (start=date, end=date, showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start=date(+time), end=date(+time), showStartTime=false,showEndTime=false,delimiter='@'}
// date to the same date range:               Apr 25, 2020                        (start===end=date(+time), showStartTime=true, showEndTime=true, delimiter='@'}
//                                                                             OR (start===end=date(+time), showStartTime=false, showEndTime=false, delimiter='@'}

// generateDateTimeLabel accepts the following arguments:
// 1. start - start point for the time period; where it may be date or date(+time) value
// 2. options - optional arguments allowing to extend function output
//    currently, the following options may be specified:
//    end - end point for the time period; where it may be date or date(+time) value
//    showStartTime - boolean flag to specify whether start timestamp is displayed if available (defaulted to true)
//    showEndTime - boolean flag to specify whether end timestamp is displayed if available (defaulted to true)
//    delimiter - delimiter to use between date and time value if both are displayed (defaulted to '@')

export function generateDateTimeLabel(start, options = {}) {
    ...
```

#### (2.7) Do not hardcode values ("magic" values). Use configuration or constants instead.

```javascript
// Bad

printLegs(legs, 4);

if (order.state === "planning") {
  printOrder(order);
}

const label = order.full_id || "NO ORDER";

// Good

// somewhere in the constants file
const PRINTED_LEGS_AMOUNT = 4;
const DEFAULT_LABEL = "NO ORDER";
const ORDER_STATES = {
  planning: "planning",
};

// ...

printLegs(legs, PRINTED_LEGS_AMOUNT);

if (order.state === ORDER_STATES.planning) {
  printOrder(order);
}

const label = order.full_id || DEFAULT_LABEL;
```

#### (2.8) Try to match returned variable names to function names.

- It makes sense if function getOranges() returns oranges rather than apples

```javascript
// Bad
const items = getLegs();
const active = getIsOrderActive(order);

// Good
const legs = getLegs();
const isOrderActive = getIsOrderActive(order);
```

#### (2.9) Try to use object destruction when you need to assign variables to the values of object properties

```javascript
// Bad
const legs = order.legs || [];
const orderState = order.state_id;

// Good
const { legs = [], state_id: orderState } = order;
```

#### (2.10) Do not overuse object destruction. Sometimes it does not makes sense to create those variables

```javascript
// Bad
const _onFormChange(name, value) => {
    const { onFormChange = () => {} } = props;

    onFormChange(DIALOG_CONST, name, value);
}

// Good
const _onFormChange(name, value) => {
    props.onFormChange(DIALOG_CONST, name, value);
}
```

#### (2.11) Name your variables with prefix "is" or "has" or "can" if the variable is a boolean

```javascript
// Bad
const orderInTransit = order.status === ORDER_STATUSES.inTransit;
const clearButton = !!component.Menu;

// Good
const isOrderInTransit = order.status === ORDER_STATUSES.inTransit;
const hasClearButton = !!component.Menu;
```

#### (2.12) Use all caps python case for configuration constants or global constants in the project

```javascript
// Bad
const apiPort = 5000;
const legStatuses = {
  dispatched: "dispatched",
  planning: "planning",
};

// Good
const API_PORT = 5000;
const LEG_STATUSES = {
  dispatched: "dispatched",
  planning: "planning",
};
```

#### (2.13) Use Object Literal Property Value Shorthand when possible

```javascript
// Bad
const id = "12345";
const status = "planning";

const order = {
  id: id,
  status: status,
  userId: getUserId(),
};

// Good
const id = "12345";
const status = "planning";

const order = {
  id,
  status,
  userId: getUserId(),
};
```

#### (2.14) Use conditional (ternary) operator for basic variable assignments instead of IF statements

```javascript
// Bad
let currency = 0;
if (order.is_assigned) {
  currency = order.currency;
} else {
  currency = api.getOrgCurrency();
}

// Good
const currency = order.is_assigned ? order.currency : api.getOrgCurrency();
```

#### (2.15.0) Use || operator if you need to default a variable value

```javascript
// Bad
let currency = getCurrency(order);
if (!currency) {
  currency = DEFAULT_CURRENCY;
}

// Good
const currency = getCurrency(order) || DEFAULT_CURRENCY;
```

#### (2.15.1) Use || operator if there is a priority value assignment

```javascript
// Good
const currency =
  order.currency || customer.currency || org.currency || DEFAULT_CURRENCY;
```

#### (2.16) Use ?? Nullish Coalescing operator if empty string, 0 or false are valid values

```javascript
// Bad
const pagination =
  query.pagination !== undefined ? query.pagination : DEFAULT_PAGINATION;

// Good
const pagination = query.pagination ?? DEFAULT_PAGINATION;
```

#### (2.17) Try to use simple conditional (ternary) operators. Try to avoid nesting them into one statement.

```javascript
// Bad
const currency = order.is_assigned
  ? order.currency
  : org.currency || DEFAULT_CURRENCY;

// Good
let currency = order.is_assigned ? order.currency : org.currency;
currency = currency || DEFAULT_CURRENCY;

// Bad
const currency = order.is_assigned
  ? order.currency
  : org.is_active
  ? org.currency
  : DEFAULT_CURRENCY;

// Good
const orgCurrency = org.is_active ? org.currency : DEFAULT_CURRENCY;
const currency = order.is_assigned ? order.currency : orgCurrency;
```

#### (2.18) Avoid using conditional (ternary) operators when logic could be simplified or re-written with || operator

```javascript
// Bad
const currency = order.org_currency || undefined;

// Good
const { org_currency: currency } = order;

// Bad
print(
  props.order_delivery_start_at
    ? toDateString(props.order_delivery_start_at)
    : ""
);

// Good
print(toDateString(props.order_delivery_start_at) || "");
```

#### (2.19) Avoid using ternary operator to find min and max values out of two given ones. Use Math.min or Math.max instead

```javascript
// Bad
salaryIncrease = salaryIncrease > MIN_INCREASE ? salaryIncrease : MIN_INCREASE;

// Good
salaryIncrease = Math.max(salaryIncrease, MIN_INCREASE);

// Bad
numOfthreads =
  numOfthreads > MAX_ALLOWED_NUM_THREADS
    ? MAX_ALLOWED_NUM_THREADS
    : numOfthreads;

// Good
numOfthreads = Math.min(numOfthreads, MAX_ALLOWED_NUM_THREADS);
```

#### (2.20) Use Array.includes() method if you need to compare string variable against multiple values

```javascript
// Bad
if (
  order.state === ORDER_STATES.planning ||
  order.state === ORDER_STATES.available ||
  order.state === ORDER_STATES.draft
) {
  doSomething();
}

// Good
if (
  [ORDER_STATES.planning, ORDER_STATES.available, ORDER_STATES.draft].includes(
    order.state
  )
) {
  doSomething();
}
```

#### (2.21) Give descriptive and meaningful names to your variables

```javascript
// Bad
const ec = getEstimatedCost();
const t = getEstimatedRevenue();
const em = (er - t) / t;

// Good
const estimatedCost = getEstimatedCost();
const estimatedRevenue = getEstimatedRevenue();
const estimatedMargin = (estimatedRevenue - estimatedCost) / estimatedCost;
```

#### (2.22) Do not use "var". Use "let" or "const" instead

- ` var`` is getting replaced with  `let/const` in ES6.

- `let` allows you to declare variables that are limited in scope to the block, statement, or expression on which it is used. This is unlike the var keyword, which defines avariable globally, or locally to an entire function regardless of block scope

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let

#### (2.23) Use string templating where it makes code more readable

```javascript
// Bad
const result =
  "(" + numOrders + ') Orders with status "' + filterOrderStatus + '" .';

// Good
const result = `(${numOrders}) Orders with status "${filterOrderStatus$}" .`;
```

#### (2.24) Use existing for..of loop when you need to loop through array elements

- New for..of loop is the fastest loop optimized for speed. It also makes your code more readable.

```javascript
// Bad
for (let i = 0, len = orders.length; i < len; i++) {
  console.log(orders[i]);
}

// Good
for (const order of orders) {
  console.log(order);
}

// if you need index
for (const [index, order] of orders.entries()) {
  console.log(index, " ", order);
}
```

#### (2.25) Avoid using Array.forEach() for looping or data permutation. Use it for executing a function per item instead

- Context: There was the time when jQuery polyfills were moving into JavaScript and forEach() was one of them. Unfortunately it is a very confusing function. Even though it is one of the built in Array function it does not really return a permutation of the array like filter() or map() or reduce() do. But if you want to loop through elements then `for..of` or `for..in` does a way better job; easier
  to read and more performant.

```javascript
// Bad
let array = [];
items.forEach((item, index) => {
  array.push({ ...item, index });
});

// Good
array = items.map((item, index) => ({ ...item, index }));
```

#### (2.26) Use Array.forEach() to execute logic for every element. But be sure to explicitely pass the element into a function since "forEach" passes 2 arguments into its argument function: "element" and "index".

- To avoid future errors when "index" passed when function expects a second argument it is recommended to follow the example.

```javascript
// Bad
for (let item in items) {
  displayElement(item);
}

// Good
items.forEach((item) => displayElement(item));
```

#### (2.27) Give descriptive names for loop variables. Try to match those names to the array variable name.

- Those extra saved characters add to the project absolutely nothing. But full names definitely make things more readable and easy to follow.

```javascript
// Bad
const filteredAccessorials = accessorials.filter((a) => !a.isPickup);

// Good
const filteredAccessorials = accessorials.filter(
  (accessorial) => !accessorial.isPickup
);
```

#### (2.28) Return early

- Use "return", "continue" and "break" to stop code execution as early as possible. This approach makes code simpler to understand, will yield in less IF statements and more sequential code.

```javascript
// Bad
const setDriver = (order = {}, driver = {}) => {
  if (driver.active) {
    if (!order.driver_id) {
      const { legs = [] } = order;
      for (let leg of legs) {
        if (leg.status !== LEG_STATUSES.dispatched) {
          api.updateLeg(leg);
        }
      }
    } else {
      onShowMessage("Order is already assigned to a driver!");
    }
  } else {
    onShowMessage("Driver is not active!");
  }
};

// Good
const setDriver = (order = {}, driver = {}) => {
  if (!driver.active) {
    onShowMessage("Driver is not active!");
    return;
  }

  if (order.driver_id) {
    onShowMessage("Order is already assigned to a driver!");
    return;
  }

  const { legs = [] } = order;
  for (let leg of legs) {
    if (leg.status === LEG_STATUSES.dispatched) {
      continue;
    }

    api.updateLeg(leg);
  }
};

// Even better
const setDriver = (order = {}, driver = {}) => {
  if (!driver.active) {
    onShowMessage("Driver is not active!");
    return;
  }

  const { driver_id: driverId, legs = [] } = order;

  if (driverId) {
    onShowMessage("Order is already assigned to a driver!");
    return;
  }

  legs
    .filter((leg = {}) => leg.status !== LEG_STATUSES.dispatched)
    .forEach((leg) => api.updateLeg(leg));
};
```

#### (2.29) Use promise chaining if your promises could be handled using the same catch() method in case of an error

- You want to have less "christmas tree" effect in your code with nested brackets and tabs. Moreover you will be able to re-use the same catch handler instead of repeating it for every promise. It is also a good starting point for possible code refactor with async await.

```javascript
// Bad
api
  .getOrder(orderId)
  .then((resp = {}) => {
    const { order = {} } = resp;

    api
      .getLegs(order.id)
      .then((resp = {}) => {
        const { legs = [] } = resp;

        updateUI(legs);
      })
      .catch((error) => {
        onShowMessage(`There was an error calling API: ${error}`);
      });
  })
  .catch((error) => {
    onShowMessage(`There was an error calling API: ${error}`);
  });

// Good
api
  .getOrder(orderId)
  .then((resp = {}) => {
    const { order = {} } = resp;

    return api.getLegs(order.id);
  })
  .then((resp = {}) => {
    const { legs = [] } = resp;

    updateUI(legs);
  })
  .catch((error) => {
    onShowMessage(`There was an error calling API: ${error}`);
  });
```

#### (2.30) Try to have as few function calls as possible in your code

```javascript
// Bad
const onValidate = (data = {}) => {
    if (!data.name) {
        this.props.onShowToastError('Organization Name is missing');
        return;
    } else if (!data.address1) {
        this.props.onShowToastError('Address 1 Line is missing');
        return;
    } else if (!data.shortCode) {
        this.props.onShowToastError('Short Code is missing');
        return;
    }

    return true;
}

// Good
const onValidate = (data = {}) => {
    let message;
    if (!data.name) {
        message = 'Organization Name is missing';
    } else if (!data.address1) {
        message = 'Address 1 Line is missing';
    } else if (!data.shortCode) {
        message = 'Short Code is missing');
    }

    if (message) {
        this.props.onShowToastError(message);
    }

    return !message;
}
```

#### (2.31) Try to avoid using destruction and defaulting in the function arguments

- It is more readable once you can see what objects are expected as function arguments. Also, a long list of properties, property name changes and default values can make function signature pretty unreadable.

```javascript
// Bad
const updateUI = (
  { active: isOrderActive, legs = [] },
  { active: isLegActive }
) => {
  if (!isOrderActive || !isLegActive) {
    return;
  }
  doSomething(legs);
};

// Good
const updateUI = (order = {}, leg = {}) => {
  if (!order.active || !leg.active) {
    return;
  }

  const { legs = [] } = order;

  doSomething(legs);
};

// This is good
const printMessage = ({
  isCapital = false,
  isRed = false,
  hasSpecialCharacters = true,
} = {}) => {
  // logic ...
};

// This is also good
const printMessage = (args = {}) => {
  const {
    isCapital = false,
    isRed = false,
    hasSpecialCharacters = true,
  } = args;

  // logic ...
};
```

#### (2.32) If a function has more than 2-3 arguments consider passing an object instead

- This approach allows you to add more variables down the road without modifying every single line where function is being used. Furthermore, you can pass only a specific set of arguments into the function since you are no longer required to pass arguments in a specified sequence.

```javascript
// Bad
const updateUI = (isActive, order, component = {}, isUserAdmin = false) => {
  // logic ...
};

// Good
const updateUI = ({
  isActive,
  order,
  component = {},
  isUserAdmin = false,
} = {}) => {
  // logic ...
};
```

#### (2.33) Try to avoid useless function wrappers

- It is an extra useless function call in the execution chain.

```javascript
// Bad
const args = {
  orderId: this.props.orderId,
  type: this.props.type,
  onSuccess: (resp) => this.props.onSuccess(resp),
};

// Good
const args = {
  orderId: this.props.orderId,
  type: this.props.type,
  onSuccess: this.props.onSuccess,
};

// Bad
const onSelectOrder = (orderId) => {
  this.props.onSelectOrder(orderId);
};

// Good

// You do not need onSelectOrder() at all. Just use this.props.onSelectOrder() where it is needed directly
```

#### (2.25) Try to name your arguments to simplify the code where possible

```javascript
// Bad
const updateUI = (item) => {
  this.setState({ selectedItem: item });
};

// Good
const updateUI = (selectedItem) => {
  this.setState({ selectedItem });
};
```

#### (2.26) Always ensure that you default arrays and objects if you are accessing their properties

- To make code extra safe and avoid JavaScript error which would stall the whole website.

```javascript
// Bad
const updateUI = (order) => {
  if (!order.active) {
    return;
  }

  console.log(`Order has ${order.legs.length} Legs`);
};

// Good
const updateUI = (order = {}) => {
  if (!order.active) {
    return;
  }

  const { legs = [] } = order;

  console.log(`Order has ${legs.length} Legs`);
};
```

#### (2.27) Use optional chaining when accessing property deep in a chain to avoid extracting a property of an undefined value

```javascript
// Bad
export const getProblems = (state) =>
  state.partner.order.orderProblems.problems;

// Good
export const getProblems = (state) =>
  state?.partner?.order?.orderProblems?.problems;

// Bad
const updateUI = (order) => {
  if (!order.active) {
    return;
  }

  console.log(`Order has ${order.legs.length} Legs`);
};

// Good
const updateUI = (order = {}) => {
  if (!order.active) {
    return;
  }

  const { legs = [] } = order;

  console.log(`Order has ${legs.length} Legs`);
};

// Also Good
const updateUI = (order) => {
  if (!order?.active) {
    return;
  }

  console.log(`Order has ${order?.legs?.length} Legs`);
};

// Better since it helps to understand what types you are dealing with
const updateUI = (order = {}) => {
  if (!order?.active) {
    return;
  }

  const { legs = [] } = order;

  console.log(`Order has ${legs.length} Legs`);
};
```

#### (2.28) When possible, avoid using optional chaining when trying to call a method which might not exist

```javascript
// Bad
const doSomething = (onClose) => {
  const arg = "something";

  if (onClose) {
    onClose(arg);
  }
};

// OK-ish
const doSomething = (onClose) => {
  const arg = "something";

  onClose?.(arg);
};

// Better
const doSomething = (onClose = () => {}) => {
  const arg = "something";

  onClose(arg);
};
```

#### (2.29) Use optional chaining when when accessing properties with an expression

```javascript
// Bad
const leg = getLeg();
const address1 = leg[`${prefix}address_1`]);

// Good
const leg = getLeg() || {};
const address1 = leg[`${prefix}address_1`]);

// Also Good
const leg = getLeg();
const address1 = leg?.[`${prefix}address_1`]);
```

#### (2.30) Always use optional chaining for accessing deeply nested hardcoded constants

- Use only when an object has more than 1 layer. No need to default the top layer. Even if the top layer is undefined nothing will break because optional chaining will handle `undefined?.undefined`.

```javascript
// Bad
const _onManifestClick = (manifestId) => {
  this.props.history.push(format(URLS.admin.manifest.page, manifestId));
};

// Good
const _onManifestClick = (manifestId) => {
  this.props.history.push(format(URLS.admin?.manifest?.page, manifestId)); // even if admin is undefined nothing breaks
};
```

#### (2.31) As much as possible, try NOT to use optional chaining for top layers of an object, instead try to use default parameters

- Our objects are mostly flat. If an object is reused in many places default parameters will default the object's top layer for the whole function scope.
- This might greatly reduce the number of times you'll have to use optional chaining.
- Although default parameters do not default valid values like `null`, our service request layer replaces all `null` values with `undefined`.
- This means that we should be relatively safe even if we just use default parameters.
- Keep in mind that if something is already defaulted with default parameters there is absolutely no need to default it again with optional chaining.

```javascript
// Bad
const extendedLegs = legs.map((leg) => ({
  ...leg,
  isOnManifest: !!leg?.master_trip_id,
  statusLabel: getStatusLabel(leg?.status_id),
  isOrderDelivered: isOrderDelivered(leg?.order),
  orderId: leg?.order?.id,
}));

// Good
const extendedLegs = legs.map((leg = {}) => ({
  ...leg,
  isOnManifest: !!leg.master_trip_id,
  statusLabel: getStatusLabel(leg.status_id),
  isOrderDelivered: isOrderDelivered(leg.order),
  // notice that we still have to use optional chaining for everything that is located deeper than the first level
  orderId: leg.order?.id,
}));
```

#### (2.32) Use async/await syntax with promises

- Async/Await is just a syntatic sugar on top of promises in JavaScript, but it can make a significant impact on making your async code more readable.

```javascript
// Bad
const printUser = (id) => {
  ApiService.getUser(id)
    .then((user) => console.log(user))
    .catch((err) => console.log(err));
};

// Good
const printUser = async (id) => {
  try {
    const user = await ApiService.getUser(id);
    console.log(user);
  } catch (err) {
    console.log(err);
  }
};
```

#### (2.33) Use await-of helper to avoid writing try catch blocks with async/await

- Not only it prevents us from writing big try catch blocks, but also makes our error handling similar to GoLang that has a multi-value return

```javascript
// Bad
const actions = {
  onGetCustomer: customerId => dispatch => {
      const promise = ApiService.find(customerId);

      promise
          .then(resp => {
              dispatch({
                type: types.GET_CUSTOMER_SUCCESS,
                customerOptions: resp.customer,
              });
          })
          .catch(error => {
              dispatch({
                type: types.GET_CUSTOMER_ERROR,
                error,
              });
          });

      return promise;
    },
}

// Good
const actions = {
  onGetCustomer: customerId => async dispatch => {
      const [resp = {}, error] = await of(ApiLocationService.find(customerId)

      if (error) {
          dispatch({
            type: types.GET_CUSTOMER_SUCCESS,
            error,
          });
          //Throwing an error to reject the promise
          throw error;
      }

      const {locations: customerOptions} = resp;

      dispatch({
        type: types.GET_CUSTOMER_ERROR,
        customerOptions,
      });

      //Resolving the promise with response returned from api request
      return customerOptions;
  },
}

```

#### (2.34) Do not use async without await

- Functions marked async must contain an await keyword unless the function expected to return a Promise by design

```javascript
// Bad
const wierdFunction = async() => {
    const result = doSomeComputation();
    //Now your function returns a promise that resolves to 'result' instead of regular return
    //Always use async when you actually need to wait for something
    return result;
}

// Good
const goodFunction = async() => {
    const value = await getSomeValueFromBackend();
    //UseSomeValue and doSomethingElse will not fire until you get your result
    useSomeValue(value);
    doSomethingElse(value);
}

// In case when you know that promise is expected ->

// Regular syntax
const doSomething = (arg) => {
   return new Promise((resolve, reject) => {
       if (!arg) {
           reject('Error');
       }

       resolve(arg);
   };
}

// Equivalent with async
const doSomething = async (arg) => {
   if (!arg) {
      throw 'Error';
   }

   return arg;
}
```

#### (2.35) Don't await unrelated API calls in the same function

- If two or more API calls do not rely on each other we want to wait for them concurrently.

```javascript
// Bad
const _onGetData = async (orderId, customerId) => {
  let error;
  [, error] = await getOrder(orderId); // takes 3s
  if (error) {
    showToastError("Failed to fetch order");
  }

  [, error] = await getCustomer(customerId); // takes 2s
  if (error) {
    showToastError("Failed to fetch customer");
  }
  // total wait time to render the full page ~5s
};

// Good

// If you need to run some logic right after the call (e.g. show a toaster) you can always wrap it in a separate function.
const _onGetOrder = async (orderId) => {
  const [, error] = await getOrder(orderId);
  if (error) {
    showToastError("Failed to fetch order");
  }
};

const _onGetCustomer = async (customerId) => {
  const [, error] = await getCustomer(customerId);
  if (error) {
    showToastError("Failed to fetch customer");
  }
};

// In this case both api calls are being dispatched concurrenly
const _onGetData = (orderId, customerId) => {
  _onGetOrder(orderId); // takes 2s
  _onGetCustomer(customerId); // takes 3s
  // total wait time to render the full page ~3s
};

// Also good

// If you need the return values in the same function and API calls don't rely on each other, you can use Promise.allSettled().
// If you you'd like to immediately reject upon any of the promises rejecting, you can use Promise.all().
// Providing Promise.all example for simplicity, please look into Promise.allSettled API if you need a safer option.
const _onGetData = async (orderId, customerId) => {
  const [resps = [], error] = await of(
    Promise.all([getOrder(orderId), getCustomer(customerId)])
  );
  if (error) {
    showToastError("Failed to fetch data");
  }

  const [order = {}, customer = {}] = resps;
  // do some logic with two payloads
};
```

### (2.36) Avoid using the `any` type when possible

- Using the any type in TypeScript essentially disables static type checking for a particular variable or expression. While it can be tempting to use any when dealing with dynamic or unknown data types, there are several reasons why you should generally avoid using the any type including loss of type saftey, code readability, and complier optimization

```typescript
// Bad
const value: any = "Hello, TypeScript!";
console.log(value.length); // No type-checking, potential runtime error

// Good
const value: string = "Hello, TypeScript!";
console.log(value.length); // Type-checking ensures 'length' exists for strings
```

### (2.37) Use absolute imports where possible

```typescript
// Bad
import { Button } from "../../common/Button";

// Good
import { Button } from "common/Button";
```

## 3. CSS / Styling

#### (3.1) Use `@emotion/styled` for styling components

We are using [`@emotion/styled`](https://emotion.sh/docs/styled) for styling our components. For consistency, debugging improvements, and to avoid CSS collisions, please use `@emotion/styled` for all new components.

- Most `classic` components are styled using BEM (Block-Element-Modifier) stylesheets, leveraging CSSGrid or rr-row/rr-column classes to structure the page and to create "structure
  tables". This patter is now deprecated and new components moving forward should use `@emotion/styled`.

- In some cases, you may need to use BEM to style your components. In those cases, please still wrap your component in `@emotion/styled` and use BEM within to style the component. This will help avoid CSS collisions.

#### (3.2) Use Design Tokens for colors, spacing, border radius, border width, opacity, shadow, font etc.

- To ensure consistency across the platform, please use design tokens from either globlal, reference or both.

`import { global, reference } from '@roserocket/design.tokens';`

- Example 1: to use the color `red` from the design tokens, you would use `color: ${global.reference.color.content.accent.red};`
- Example 2: to use add a `4.0rem` padding to a div, you would use `padding: ${global.reference.spacing.xxl};`

#### (3.2.1) If you need to use a value that is not available as a design token, use rem units instead of px when possible

- Double check with your designer, post in #g-frontend or #design-talk if you are unsure. The team may want to update our design tokens if the value is used in multiple places!

#### (3.3) Leverage our [Bit component library](https://bit.cloud/roserocket/design) over Chromatic / Storybook components when available.

- All modern components are being migrated from Chromatic / Storybook to Bit.dev. Before using a component, please check if it has a Bit.dev version available. You can see all available components here: https://bit.dev/roserocket/roserocket-design or by looking through the [`rr-ui-components` repository](https://github.com/RoseRocket/rr-ui-components/tree/main/bit)
- Example: A commonly seen mistake is using `@roserocket/Typography` instead of the new bit component `@roserocket/design.Typography` over

#### (3.3.1) When importing a component from Bit.dev, do not include `/dist/...` in the import path

- Example: `import TextInput from '@roserocket/design.text-input'` instead of: `import TextInput from '@roserocket/design.text-input/dist/text-input'`

#### (3.4) Avoid using !important in your CSS rules

- `!important` is an anti-pattern in CSS world. It adds a dimension of "magic" to your code that can lead developers to a rabbit-hole trying to find why an element is styled one way even though CSS is clearly defined. It can get even harder to debug and manage when multiple `!important` rules are starting to overlap. For clarity purposes, it is best to avoid using it as much as possible.

#### (3.5) Avoid using inline styles

```typescript
// Bad

<div style="padding: ${global.spacing.lg}"> Some content </div>

// Good
 const MyStyledDiv= styled.div`
  padding: ${global.spacing.lg};
`;
 ...
// Usage
 <MyStyledDiv> Some content </MyStyledDiv>

```

#### (3.6) Avoid using four-properties shorthands

- This type of shorthands is difficult to memorize and notice while debugging which often causes unintentional bugs in css rules

```typescript
// Bad
 const MyStyledContainer = styled.div`
  padding: 4rem 3rem 0 1rem;
`;

// Good
const MyStyledContainer = styled.div`
  padding-top: 4rem;
  padding-right: 3rem;
  padding-left: 1rem;
```

#### (3.7) When programming with z-index refer to the enum Z_INDEXES before hardcoding the value

```typescript
//Bad
const StyledCurrencySection = styled(StyledSection)`
  max-width: 242px;

  .selectInput__menu {
    z-index: 900;
  }
`;
// Good
const StyledCurrencySection = styled(StyledSection)`
  max-width: 242px;

  .selectInput__menu {
    z-index: ${Z_INDEXES.popover};
  }
`;
```

## 4. React

#### (4.1) Functions internal to a component should be prefixed with '\_'. When possible prefix with `on` for additional clarity

```javascript
// Bad
const OrderDetailsHeader = () => {
  const namedFormChange = (name) => {
    /* ... */
  };
  const getOrder = (id) => {
    /* ... */
  };

  return <JSX />;
};

// Good
const OrderDetailsHeader = () => {
  const _onNamedFormChange = (name) => {
    /* ... */
  };
  const _onGetOrder = (id) => {
    /* ... */
  };

  return <JSX />;
};
```

#### (4.2) Try to keep variable names the same which you use to pass into component properties.

```javascript
// Bad
const OrderDetailsHeader = () => {
  const namedFormChange = (name) => {
    /* ... */
  };
  const _onHandleOrder = (id) => {
    /* ... */
  };

  const items = getCustomers();

  return (
    <Header
      customers={items}
      onNamedFormChange={namedFormChange}
      onGetOrder={_onHandleOrder}
    />
  );
};

// Good
const OrderDetailsHeader = () => {
  const _onNamedFormChange = (name) => {
    /* ... */
  };
  const _onGetOrder = (id) => {
    /* ... */
  };

  const customers = getCustomers();

  return (
    <Header
      customers={customers}
      onNamedFormChange={_onNamedFormChange}
      onGetOrder={_onGetOrder}
    />
  );
};
```

#### (4.3) When using defaults for functions, object and arrays in props, reference them to a local const (`const emptyObject = {};`, `const emptyArray = [];`, `const emptyFunction = () => {};`) instead of `{}`, `[]` or `() => {}` to avoid unnecessary re-renders.

```typescript
// Bad
const OrderDetailsHeader = ({
  order = {},
  org = {},
  someArray = [],

  onClick = () => {},
  onBlur = () => {},
}) => {
  return <div onClick={onClick}>{order.title}</div>;
};

// Good
const emptyObject = {};
const emptyArray = [];
const emptyFunction = () => {};

const OrderDetailsHeader = ({
  order = emptyObject,
  org = emptyObject,
  someArray = emptyArray,

  onClick = emptyFunction,
  onBlur = emptyFunction,
}) => {
  return <div onClick={onClick}>{order.title}</div>;
};
```

#### (4.4) Write variables first and then functions

```javascript
// Bad
<OrderDetailsHeaderActions
    onStartClaim={_onStartClaim}
    isEditable={isEditable}
    onCreateOrderProblem={_onCreateOrderProblem}
    isCreatingQuotes={isCreatingQuotes}
    availableActions={availableActions}
    onCreateQuotes={_onCreateQuotes}
/>

// Good
<OrderDetailsHeaderActions
    isEditable={isEditable}
    isCreatingQuotes={isCreatingQuotes}
    availableActions={availableActions}
    onStartClaim={_onStartClaim}
    onCreateOrderProblem={_onCreateOrderProblem}
    onCreateQuotes={_onCreateQuotes}
/>
```

#### (4.4.1) Keep boolean args first. True can be passed simply by specifying the name of the property

```javascript
// Bad
<OrderDetailsHeaderActions
    onStartClaim={_onStartClaim}
    isEditable={true}
    onCreateOrderProblem={_onCreateOrderProblem}
    isCreatingQuotes={false}
    availableActions={availableActions}
    onCreateQuotes={onCreateQuotes}
/>

// Good
<OrderDetailsHeaderActions
    isEditable
    isCreatingQuotes={false}
    availableActions={availableActions}
    onStartClaim={_onStartClaim}
    onCreateOrderProblem={_onCreateOrderProblem}
    onCreateQuotes={onCreateQuotes}
/>
```

#### (4.5) Avoid spreading variables into components. Be explicit with what is passed in

```javascript
// Bad
<OrderRow
    {...otherProps}
    key={order.id}
    onClick={_onClick}
/>

// Good
<OrderRow
    arg1={otherProps.arg1}
    arg2={otherProps.arg2}
    key={order.id}
    onClick={_onClick}
/>
```

#### (4.6) Consider splitting your component into multiple smaller ones if you find that component is growing to be too big or you have many IF statements within.

- Smaller components are more re-usable and easier to extend if you need to change things up. Less nested IF components lead to a more readable and manageable code as well.

#### (4.7) Try to create components as generic as possible. Avoid hardcoded business workflows/APIs within subcomponents; use a Wrapper component for business logic, states and connections and pass down only the necessary props.

```javascript
// Bad
const OrderActionComponent = ({ order = defaultObj }) => {
  return <StyledOrderActionPill label={order.status} onClick={onCallSomeAPI} />;
};

// Good
const OrderActionPill = ({ className, label = "", onClick = defaultFunc }) => {
  return <StyledPill label={label} onClick={onClick} />;
};

const OrderActionComponent = ({ order = defaultObj }) => {
  return <OrderActionPill label={order.status} onClick={_onCallSomeAPI} />;
};
```

#### (4.7.1) Avoid passing business logic data models into smaller or generic components. Try to pass basic value properties (unless it is a component wrapper).

```javascript
// Bad
const RevenueItem = ({ order = defaultObj }) => {
  return (
    <div>
      <span>{order.revenue}</span>
      <span>{order.exchangeRate}</span>
    </div>
  );
};

// Good
const RevenueItem = ({ revenue, exchangeRate }) => {
  return (
    <div>
      <span>{revenue}</span>
      <span>{exchangeRate}</span>
    </div>
  );
};
```

#### (4.8) Allow developers to overwrite default component behavior if necessary.

- There are times when developers need to overwrite or enhance the components' default behavior, even though the element is designed to do a specific thing. This approach makes code much more flexible and guides developers towards proper code writing instead of spawning multiple IF statements or extra arguments within the component.

```javascript
// Bad
const LegStatusPill = ({ someArg }) => {
  return <StyledPill hasToolTip onClick={onDefaultBehavior} />;
};

// Good
const LegStatusPill = ({
  someArg,
  hasToolTip = true,
  onClick = onDefaultBehavior,
}) => {
  return <StyledPill hasToolTip={hasToolTip} onClick={onClick} />;
};
```

#### (4.9) Do not use {} in the code for string which do not require computations

```javascript
// Bad
<OrderRow data-test={`rr-orderRow`} />


// Good
<OrderRow data-test="rr-orderRow" />
```

#### (4.10) Avoid using Inline Anonymous Functions. Try to use component functions instead and try to match the name of functions

- We are no longer doing it for performance purposes but for readability purposes. Keeping structure of all components the same and easy way to read JSX.

```javascript
// Bad
<MenuItem
  key={MENU_ITEM_ACTIONS.downloadOrdersPDF}
  onClick={() => props.onExport(id)}
/>;

// Good
const _onClick = () => {
  props.onExport(id);
};

<MenuItem key={MENU_ITEM_ACTIONS.downloadOrdersPDF} onClick={_onClick} />;
```

## 5. React Hooks

#### (5.1) Do not use React.memo() for every component. Only use it for components which truly need performance optimization OR this performance is visible enough to the user OR could be traced through profiling tools.

- For more information, checkout the answer on ["#14000 Make stateless components "Pure" by default in React 17"](https://github.com/facebook/react/issues/14000)

#### (5.1.1) DO use "useMemo" when trying to use the same "useSelector" across multiple components

- Do not forget to use "shallowEqual" if returning value is an object or an array

- For more information check out: [https://react-redux.js.org/next/api/hooks#useselector-examples](https://react-redux.js.org/next/api/hooks#useselector-examples)

```javascript
// DriverCard.js
import React, { useMemo } from "react";
import { shallowEqual, useSelector } from "react-redux";
import { makeDriverSelector } from "./selectors";

const DriverCard = ({ userProfileId }) => {
  const driverSelector = useMemo(makeDriverSelector, []);
  const driver = useSelector(driverSelector(userProfileId), shallowEqual) || {};
};

// selectors.js
export const makeDriverSelector = () => (userProfileId) =>
  createSelector([getManifestAssignees], (manifestAssignees = []) => {
    // logic using userProfileId
  });
```

#### (5.1.2) You need to use "useCallback()" if you pass callbacks into memo-ized component.

- Supporting context: [https://dmitripavlutin.com/use-react-memo-wisely/#4-reactmemo-and-callback-functions](https://dmitripavlutin.com/use-react-memo-wisely/#4-reactmemo-and-callback-functions)

```typescript
// Bad
const Parent = () => {
    return <MemoChild var="1" onClick={() => sayHello()} />
};

// Bad
const Parent = () => {
    const _onClick = () => sayHello();
    return <MemoChild var="1" onClick={_onClick}/>
};

// Good
import { useCallback } from 'react';

const Parent = () => {
    const _onClick = useCallback(() => sayHello(), [])

    return <MemoChild var="1" onClick={_onClick}>
};
```

#### (5.2) Import React from "react" in every file where you use JSX. This import should be first in the file.

```typescript
// Bad
import { someFunction } from "some-library";
import React, { useState } from "react";

// Good
import React, { useState } from "react";
import { someFunction } from "some-library";
```
