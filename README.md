# model package spec (WIP)

Our model package specification.

## Why?
After months working with meteor on a big project you learn that packaging is the best way available to keep things under control. Model is one of the key parts of almost any application and we decided to share how we structure our model packages. There are many reasons on why we build it this way, some of them are:

- Things move fast (really fast, too fast) so we should keep parts of the app as independent as possible so in the future changing a part won’t kill you.
- Testing in meteor is a pain, at least until meteor 1.3 is launched and we have a more standard way of testing.
- Meteor have many features perfect for prototyping or small projects but many of them should be avoided in a big project.

## Principles

Our way of packaging model is based on some ideas that drive our code:

- **Predictable:** It should be easy to see what happened and what would happen.
- **Testable:** It is easy to unit test 100% your code.
- **Understandable:** It should be easy to code and read.
- **Encapsulated:** Model should only expose what the application needs. Database operations, methods… all should be encapsulated inside the package and never used outside it.
- **Meteorized:** We’re using meteor for a reason. Model should be based on Latency compensation.
- **Secure:** Client database operations are bad, always. Any operation from client should be blocked except inserts where you need id immediately after insert.
- **Functional:** Your model should represent the data you save into database but should’t contain methods. Functions never mutate data. This makes it easy to read and test.


## Changes

This spec is still a WIP. We've code running based on it but we're still doing changes and open to changes. We may have to do more changes once Meteor 1.3 is out and many more during 2016/1017 due to meteor expected big changes (GraphQL I'm looking at you)

## Boilerplate

We have also published the base code we use [here](https://github.com/mmmelon/boilerplate-model). Just clone, remove git and replace Base/base with your model instance name (Task/task for example) and you’re ready to start coding your model.

## Example 

You can check the basic example [here](https://github.com/mmmelon/example-model/tree/master).

## Structure

Any model package should share the same file/folder estructure. That makes it easy to understand. Also, some parts use a custom namespace. Functions declared in methods use ModelName.Methods. Check the example to get a better idea on how and why.

### source

Source folder contains all the model source code. Your code goes inside this folder

#### client

Client only code
- **model.js**: Client only model functions

#### common

Client shared between client and server
- **methods.js:** (Methods namespace) Meteor methods.Meteor methods only check parameters and call a function.Never place your code inside a meteor method.
- **model.js:** Client/Server shared functions, collection declaration and model declaration.
- **triggers.js:** (Triggers namespace) Collection hooks should be avoided. Instead we use functions that should be called when some action is done.
- **errors.js:** (Errors namespace) Error types declaration. Never return an string or code as error. You should always throw errors declared inside this file.

#### persistence

Read/Write functions. There should’t be any database query outside this folder.
- **reading.js:** (Query namespace) Model should wrap mongo reading queries. That’s the best way to make your application independent from database. Included here the mongo collection declaration too.
- **writing.js:** (Write namespace) Same idea applied to database write queries. Keep in mind that writing in client is blocked so this functions will run only inside a simulation or server side.

#### server

Only server side code.
- **model.js:** Model functions visible only to server side.

### tests

Place here your tests.
- **client.js:** Client side tests
- **common.js:** Shared tests
- **methods.js:** Test method related functions, not meteor methods.
- **stubs.js:** Needed stubs.

## Declaration

Collection and model object is declared in **common/model.js**. You can use ES6 classes or just a function to declare your model object. Use it to declare default values:

```
Task = function(task){
	this.name = task.name || “defaultName”;
}
```

We use ES6 as it makes it easier to read. Our boilerplate always include ecmascript package so you can use ES6 always.

## Reading

Reading file always includes some base functions:

- Task.Query.findAll({sort,limit,skip}): Same as Tasks.find({}). Sort, limit and skip are optional parameters using mongo standard format.
- Task.Query.onebyIdCursor({id}): Same as Tasks.find({_id:id})
- Task.Query.oneById({id}): Sames as Tasks.findOne(id)

## Writing

Writing file always includes:

- Task.Write.create({object}): Same as Tasks.insert(object)

## Errors

Errors happen, but there should be a clean way to know what an error means and a declaration reference so your application could be ready to that errors. Errors are declared in **common/errors.js** following the form:

```
Task.Errors = {
	Validation:{
		NameEmpty:”NameEmpty”
	}
}
```

With this declaration you could throw an error when name is empty but instead of using an easy to forget error string you’ll use:

```
Task.Errors.Validation.NameEmpty;
```

## Validation

Validation should be done using a function. That function should return a list of errors and is declared inside **common/model.js**. It should take the instance object as the only parameter and return an object containing a Boolean valid field and errors array in errors field.

```
Task.validate = function(task){
    var errors = [];
    if(!Match.test(task.name, String)){errors = errors.concat(Task.Errors.Validation.NameEmpty)};
    return {
        valid: errors.length === 0,
        errors:errors
    };
}
```

This function won’t be used outside model, only inside functions called from main app to perform changes in persistence layer (database).

## Testing

We decided to go with the **mocha + chai + sinon** combo for testing. We use **practicalmeteor:mocha** as our test reporter. To run your tests just run:

```
Meteor test-packages —driver-package practicalmeteor:mocha packageName -p 4000
```

We use port 4000 as main app is running on port 3000 by default.

## Functions

Javascript treats functions as first class citizens but there’s something I personally missed. Named parameters. That’s why we use this function format:

```
Task.isProjectChanged = function({oldTask,newTask}){…}
```

instead of:

```
Task.isProjectChanged = function(oldTask,newTask){…}
```

so function call is transformed from:

```
Task.isProjectChanged(task,newTask)
```

to:

```
Task.isProjectChanged({oldTask:task,newTask:newTask})
```

This removes parameter order relevance and includes parameter name in call so you always know what function requires.
