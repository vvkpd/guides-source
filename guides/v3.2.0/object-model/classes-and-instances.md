As you learn about Ember, you'll see code like `Component.extend()` and
`DS.Model.extend()`. Here, you'll learn about this `extend()` method, as well
as other major features of the Ember object model.

### Defining Classes

To define a new Ember _class_, call the [`extend()`](https://api.emberjs.com/ember/2.16/classes/@ember%2Fobject/methods/extend?anchor=extend) method on
[`EmberObject`](https://api.emberjs.com/ember/2.16/modules/@ember%2Fobject):

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  say(thing) {
    alert(thing);
  }
});
```

This defines a new `Person` class with a `say()` method.

You can also create a _subclass_ from any existing class by calling
its `extend()` method. For example, you might want to create a subclass
of Ember's built-in [`Component`](https://api.emberjs.com/ember/2.16/classes/Component) class:


```javascript {data-filename=app/components/todo-item.js}
import Component from '@ember/component';

export default Component.extend({
  classNameBindings: ['isUrgent'],
  isUrgent: true
});
```

### Overriding Parent Class Methods

When defining a subclass, you can override methods but still access the
implementation of your parent class by calling the special `_super()`
method:

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  say(thing) {
    alert(`${this.name} says: ${thing}`);
  }
});

const Soldier = Person.extend({
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ', sir!' to the variable `thing` passed in
    this._super(`${thing}, sir!`);
  }
});

let yehuda = Soldier.create({
  name: 'Yehuda Katz'
});

yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
```

In certain cases, you will want to pass arguments to `_super()` before or after overriding.

This allows the original method to continue operating as it normally would.

One common example is when overriding the [`normalizeResponse()`](https://api.emberjs.com/ember-data/2.16/classes/DS.JSONAPISerializer/methods/normalizeResponse?anchor=normalizeResponse) hook in one of Ember-Data's serializers.

A handy shortcut for this is to use a "spread operator", like `...arguments`:

```javascript
normalizeResponse(store, primaryModelClass, payload, id, requestType)  {
  // Customize my JSON payload for Ember-Data
  return this._super(...arguments);
}
```

The above example returns the original arguments (after your customizations) back to the parent class, so it can continue with its normal operations.

### Creating Instances

Once you have defined a class, you can create new _instances_ of that
class by calling its [`create()`](https://api.emberjs.com/ember/2.16/classes/@ember%2Fobject/methods/create?anchor=create) method. Any methods, properties and
computed properties you defined on the class will be available to
instances:

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  say(thing) {
    alert(`${this.name} says: ${thing}`);
  }
});

let person = Person.create();

person.say('Hello'); // alerts " says: Hello"
```

When creating an instance, you can initialize the values of its properties
by passing an optional hash to the `create()` method:

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  helloWorld() {
    alert(`Hi, my name is ${this.name}`);
  }
});

let tom = Person.create({
  name: 'Tom Dale'
});

tom.helloWorld(); // alerts "Hi, my name is Tom Dale"
```

Note that for performance reasons, while calling `create()` you cannot redefine an instance's
computed properties and should not redefine existing or define new methods. You should only set simple properties when calling
`create()`. If you need to define or redefine methods or computed
properties, create a new subclass and instantiate that.

By convention, properties or variables that hold classes are
PascalCased, while instances are not. So, for example, the variable
`Person` would point to a class, while `person` would point to an instance
(usually of the `Person` class). You should stick to these naming
conventions in your Ember applications.

### Initializing Instances

When a new instance is created, its [`init()`](https://api.emberjs.com/ember/2.16/classes/EmberObject/methods/init?anchor=init) method is invoked
automatically. This is the ideal place to implement setup required on new
instances:

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  init() {
    alert(`${this.name}, reporting for duty!`);
  }
});

Person.create({
  name: 'Stefan Penner'
});

// alerts "Stefan Penner, reporting for duty!"
```

If you are subclassing a framework class, like `Ember.Component`, and you
override the `init()` method, make sure you call `this._super(...arguments)`!
If you don't, a parent class may not have an opportunity to do important
setup work, and you'll see strange behavior in your application.

Arrays and objects defined directly on any `Ember.Object` are shared across all instances of that class.

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  shoppingList: ['eggs', 'cheese']
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.shoppingList.pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.shoppingList.pushObject('sausage');
  }
});

// Stefan and Robert both trigger their addItem.
// They both end up with: ['eggs', 'cheese', 'bacon', 'sausage']
```

To avoid this behavior, it is encouraged to initialize those arrays and object properties during `init()`. Doing so ensures each instance will be unique.

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  init() {
    this.set('shoppingList', ['eggs', 'cheese']);
  }
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.shoppingList.pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.shoppingList.pushObject('sausage');
  }
});

// Stefan ['eggs', 'cheese', 'bacon']
// Robert ['eggs', 'cheese', 'sausage']
```

### Accessing Object Properties

When accessing the properties of an object, use the [`get()`](https://api.emberjs.com/ember/2.16/classes/@ember%2Fobject/methods/get?anchor=get)
and [`set()`](https://api.emberjs.com/ember/2.16/classes/@ember%2Fobject/methods/set?anchor=set) accessor methods:


```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  name: 'Robert Jackson'
});

let person = Person.create();

person.get('name'); // 'Robert Jackson'
person.set('name', 'Tobias Fünke');
person.get('name'); // 'Tobias Fünke'
```

Make sure to use these accessor methods; otherwise, computed properties won't
recalculate, observers won't fire, and templates won't update.
