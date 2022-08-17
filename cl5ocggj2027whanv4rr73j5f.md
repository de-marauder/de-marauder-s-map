## Handling Asynchronous Operations In JavaScript


# Introduction
The word “asynchronous” basically means “out of order”. It is a concept that implies execution of tasks in a non sequential manner. It enables operations to run concurrently that is, at the same time. 

Many programming languages employ this concept and it can be used to do things like improve user experience or speed up the execution time of a script by removing wait time from their program's execution lifespan or allowing processes to run concurrently. This is typically observed when there are tasks that require significant time to execute and waiting for them would be detrimental to user experience. Tasks such as API calls, execution of large loops and server calls fit this description.

These operations are allowed to run in the background (in a manner of speaking) while the rest of the program executes without waiting for it. This can however cause some problems in programming languages like JavaScript that are synchronous by default.

In this article, we will be going over the 3 different ways asynchronicity can be handled in JavaScript:
Callback
Promises
Async/Await

## Prerequisites
Requirements to completely follow along with this tutorial
- Basic knowledge of Javascript

<br>

# Callbacks
This is a function passed into another function (called a higher order function) as a parameter to be invoked somewhere in the logic described in that higher order function. Typically after certain tasks or conditions have been met. Functions like `setTimeout()`, `setInterval()` and array methods like `map()` and `filter()` are typical examples of functions that accept callbacks.

These callbacks can be used to handle asynchronous operations by invoking them after the asynchronous operation is done. 

Suppose we had an array of users.

```js
const users = [{name: 'marauder', age: 1000}, {name: 'zeldor', age: 100}];
```
We could decide to mimic a database call to get a particular user. One way to do this would be to use the setTimeout function to delay a call on the array.
```js
let fetchedUser;
setTimeout(()=>{
	fetchedUser = users.filter((user)=>{
	    return user.name === 'marauder';
    });
}, 3000);
```

We could then proceed to perform some action on the returned user. Perhaps add a "role" property of "admin".

```js
let fetchedUser;
setTimeout(()=>{
	fetchedUser = users.filter((user)=>{
	    return user.name === 'marauder';
    });
}, 3000);

fetchedUser[0].role = 'admin';
```

This little piece of code above would not work properly. This is because at the point were the `fetchedUser` is being assigned a role `fetchedUser` would be undefined because the last line of code in this example does not wait for the `setTimeout` to finish executing but rather carries on and by the time `setTimeout` is finally done, the damage would already be done.

What this little example here demonstrates is that we need a way to tell our program to run specific blocks of code dependent on asynchronous data only when said data is available. 

Callbacks answered this problem. We can now proceed to fix the error in our code by moving the last statement into the callback function passed to `setTimeout`.

```js
let fetchedUser;
setTimeout(()=>{
	fetchedUser = users.filter((user)=>{
	    return user.name === 'marauder';
    });
fetchedUser[0].role = 'admin';
}, 3000);
```

Now our code will run perfectly. 

This provides a solution to our problem. We could handle all our asynchronous data like this using callbacks but things can get real ugly real fast and I mean this quite literally. Imagine having a function where you make an asynchronous request and use the data obtained from this request to make another and then use the data obtained from this new request to make another request whose returned data you now use to perform your operations. The code would look a bit like this,


```js
const user = {name: 'marauder', age: 1000, id: 1337};
function makeAsyncOperation () {

	makeFirstRequest(()=>{

		// make use of asyncFunc1 to get data about a user
	    asyncFunc1(`https://someurl.com/api/user/${user.id}`, (res1)=>{ 

            // make use of asyncFunc2 to get data about a user location
	    	asyncFunc2(`https://someurl.com/api/map/${res1.location}`, (res2)=>{

			    // make use asyncFunc3 to get data about a restaurants in user’s area
                asyncFunc3(`https://someurl.com/api/${res2.location}/restaurants`, (res3)=>{
	                // use the data from asyncFunc3 to update your webpage or perform some other task
                    … 
                });
           });
        })
    })
}
```

After going through the above code you should understand what I meant by “things can get real ugly real fast”. This type of code is so undesirable that a name was coined for it, “callback hell”. Not only is it ugly but it also gets difficult to follow the logic of the code especially when a lot more is going on in each callback.

Another disadvantage of this method of handling asynchronous requests is something called "control Inversion". This basically means that whenever you use a callback, you’re basically relinquishing all control rights to that callback and giving it to the higher order function trusting that it will handle the callback in a specific way. 

This becomes a problem especially when using 3rd party APIs where you have no control over the higher order functions receiving your callbacks. A 3rd party API could decide to change the way one of their functions handles its callbacks and this could have the implication of breaking your application.

So, what we need now is a way to not relinquish control of our functions and simpler syntax for handling asynchronous requests. For this we look to `Promises`.

<br>

# Promises
A promise is a built-in JavaScript object that radicalized the way asynchronous operations are handled. It operates by keeping track of the current state of an asynchronous request/operation. If the asynchronous operations is successful an in-built `resolve` function is invoked. If the operation fails or could not be completed for some reason an in-built `reject` function is invoked. 

A promise is designed to always be in one of 3 states:
- **Pending**: The default state showing that the asynchronous operation is still ongoing.
- **Fulfilled**: This state is attained when the promise resolves. It is set using the `resolve` function which is available as the first parameter in the `Promise` constructor’s callback.
- **Rejected**: This state is attained when an error occurs during the execution of the asynchronous operation. This state is set by invoking the `reject` function which is also available in the `Promise` constructor’s callback function as the second parameter.

A promise is initialized using its constructor function like so,

```js
const promise = new Promise((resolve, reject)=>{
	try {
	    // perform an asynchronous operation. 
    	setTimeout( ()=>{
            const data = "something";
            resolve(data);
        }, 5000);
    	return;
    } catch (error) {
    	reject(error);
    }
});
```

As you can see, the promise still makes use of callbacks but does so in an elegant way that eliminates the two disadvantages of handling asynchronous operations using only callbacks. 

In the above code, `promise` will evaluate to the value stored in the `data` variable which is passed to `resolve` function if the asynchronous `fetch` request is completed successfully. If the request/operation fails, `promise` returns the `error` object passed into the `reject` function.

You should note that if you write the asynchronous function yourself from scratch and it doesn't return a promise, you will need to pass in a callback so that `resolve` can be invoked in it's logic after its asynchronous operations are done. This is exactly what I did in this example. Since `setTimeout` mimics an asynchronous operation and does not return a promise, I had to invoke `resolve` in `setTimeout`'s callback.

Otherwise, if the asynchronous function you're working with already returns a promise which will most likely be the case. You can chain a `then` to it. For example if you're working with the fetch API you could do something like this.

```js
const myPromise = new Promise((resolve)=>{
    fetch('https://jsonplaceholder.typicode.com/comments?postId=1').then(data => resolve(data));
});
```

This simply hits one of the dummy endpoints of [jsonplaceholder](https://jsonplaceholder.typicode.com) and stores its response in the `data` variable. This variable will be available as a parameter to the `then` method chained to the promise's invocation.

This could be redundant though since `fetch` already returns a promise, it need not be wrapped in a promise again so a simple 

```js
fetch('https://someurl.com/api/bank/exchange').then((data)=>{
    // perfom another operation with data
    console.log(data);
}):
```
would suffice.

Another important feature of the JavaScript `Promise` which you have already seem a glimpse of in these last two examples is that its returned value is also wrapped in a promise and can therefore utilize `Promise` methods such as `then()`, `catch()` and `finally()`. This allows promises to be chained to each other. 

The `then()` method accepts two callbacks, a **promise fulfilled handler** and a **promise rejected handler**. The first handles the success of the promise and is called when the original promise is fulfilled. The second callback is called if the original promise is rejected and you wish to handle that eventuality immediately instead of waiting for the `catch()` method at the end of the promise chain. The **promise fulfilled handler** accepts parameters identical to those in the `resolve` function of the original promise and the **promise rejected handler** accepts parameters identical to those in the `reject` function.

The `catch()` method is invoked when its original promise is rejected and accepts a callback defining tasks to be performed if the promise fails. The callback accepts parameters identical to those passed into the `reject` function in the original promise.

The `finally()` method is called when the state of the promise changes from pending to any of the other two `Promise` states. It is called once the promise is done regardless of whether it succeeded or failed.

It is important to note that the return values of these methods are also wrapped in promises and can as well be chained to each other because of this.

Here’s a simple example,

```js
const myPromise = new Promise((resolve, reject)=>{
	try {
	    // perform an asynchronous operation. Perhaps try to retrieve forex data from an API.
	    asyncFunc('https://someurl.com/api/bank/exchange', (data)=>{
            resolve(data);
	        return;
        });    
    } catch (error) {
    	reject(error);
        return;
    }
});

// We can then invoke myPromise and utilize the methods explained above

myPromise().then((data)=>{
	// The data variable is equivalent to the one passed into the resolve function in myPromise.
    // Do some stuff…
}).catch((error)=>{
	// The error variable is equivalent to the one passed into the reject function in myPromise.
	// Handle error
}).finally((info)=>{
	// ‘info’ contains either the return value of the preceding ‘then’ or ‘catch’ methods
	// perform some action after the promise is done executing
})
```

Now it’s a bit clearer to see how the disadvantages of handling asynchronous operations with callbacks have been mitigated. Instead of passing your function into a third party API and relinquishing control the third party simply rewrites their API to return a Promise. This will allow you to call the API’s function which will now return a promise which you can then chain a `then` to (no pun intended). Thus, you remain in complete control of how your functions are being executed.

The example above also shows the improved legibility of the operations being performed as opposed to the `callback hell` incurred when using only callbacks.

If we wanted to rewrite the callback example using promises it would look like this code below. For simplicity, we'll be using a hypothetical function `asyncFunc` that behaves just like the `fetch` API but doesn't require the use of the `json()` method on the initial response.

```js
const user = {name: 'marauder', age: 1000, id: 1337};
const makeAsyncOperation = () => {
    return new Promise((resolve, reject)=>{
    	try {
    		// Some asynchronous operation perhaps open a connection to a server
    		resolve();
        } catch (error) {
        	// If some error occurred or some condition was not met
    		reject(error);
        }
    });
}

// We can now call makeAsyncOperation as a promise
makeAsyncOperation().then(()=>{ // makeFirstRequest
	// make use of fetch API to get data about a user

    return asyncFunc(`https://someurl.com/api/user/${user.id}`);
}).then((res1)=>{ // makeSecondRequest
    // make use of fetch API to get data about a user location 
    
    return asyncFunc(`https://someurl.com/api/map/${res1.location}`);
}).then((res2)=>{ // makeThirdRequest
    // make use of fetch API to get data about a restaurants in user’s area

    return asyncFunc(`https://someurl.com/api/${res2.location}/restaurants`);
}).then((res3)=>{
    	
    // use the data in res3 to update your webpage
    … 

}).catch((error)=>{
	console.log(error);
})

```

This is definitely easier to follow and think about but (yes, there’s still a “but”) it can still be difficult to follow what’s going on especially if you’re a beginner. Not to mention that the idea of chaining multiple methods is quite ugly. 

What we need now is a  cleaner way to handle promises and the answer is (...drumroll...). <br>
Yes!  <br>
You guessed it!  <br>
It’s `async await`. <br>

<br>

# Async Await
Along with ES6 came the ‘holy grail’ of handling asynchronicity in javascript. It’s clean, elegant and intuitive. It makes writing asynchronous javascript look synchronous. No callbacks, no promise chaining and to top it off, it’s dead simple to implement.

Before we delve into examples, there are a couple of rules you must be aware of in order to use `async await` properly. They include:
1. To mark a function as async use any of these syntax:
    - `async function asyncFunctionName () {}`
    - `const asyncFunctionName = async () => {}`
    - `const asyncFunctionName = async function() {}`
    -  Or wrap any of the above functions in an IIFE (Immediately Invoked Function Expression) if you find no need to assign a name to the function and want to invoke it at once immediately after it is defined. <br> `(async ( ) => { })()`

2. The `await` keyword can only be used in a function marked as `async`. Javascript's engine will throw an error if you don’t adhere to this rule.
3. The `await` keyword only works on functions that return a promise (yes, we’re still talking about promises) and will be ignored if appended on a function that does not.<br>
Syntax: `const returnedValue = await promiseFunction();`

Using `async await` to rewrite the asynchronous code handled with callbacks above, the code would look like this,

```js
const user = {name: 'marauder', age: 1000, id: 1337};
const makeAsyncOperation = async () => {
	try {
	    // Some asynchronous operation
    	await openServerConnection(); // await the completion of an asynchronous operation
    } catch (error) {
    	// If some error occurred or some condition was not met
    	console.log(error);
    	throw new Error(error);
    }
}

// We can now call makeAsyncOperation using async await

( // wrap all the operations inside an IIFE and mark it as async
    async () => {
        await makeAsyncOperation();  // opens server connection
    	const res1 = await fetch(`https://someurl.com/api/user/${user.id}`);
    	const res2 = await fetch(`https://someurl.com/api/map/${res1.location}`);
    	const res3 = await fetch(`https://someurl.com/api/${res2.location}/restaurants`);
})()

// use the data in res3 to update your webpage

```

Now,  there seems to be a lot going on here but it’s really not that complicated at all. The first thing we did was to create the `makeAsyncOperation` function. Notice how it’s marked with the keyword `async`? 

`async` is a reserved keyword in javascript that basically informs the javascript engine that a particular function will be performing asynchronous operations.

Inside the `makeAsyncOperation` function we invoke our asynchronous operation using a try-catch block and throw an error if any occurs. This eliminates the need for a `catch` method to be chained. The `openServerConnection` function is a hypothetical function that returns a promise and as such can be awaited.

We then create an IIFE marked as async and invoke all our asynchronous `fetch` functions which are also awaited because they also return a promise. 

At this point I should point out that the `await` keyword simply waits for the returned value of the promise when it gets fulfilled. Thus, we are able to obtain `res1`, `res2` and `res3` in a seemingly synchronous manner. This by all counts is awesome and is currently the best way to handle asynchronicity in javascript.

So there you have it, the three methods for handling asynchronous operations in javascript.

<br>

# Conclusion
Asynchronicity is a very important concept in programming. It enables a program to run multiple tasks at the same time. However, this could lead to problems when it takes too long for a piece of asynchronous code to execute. This could cause your application to break because your asynchronous code returns a value after your application has already finished running. This could occur in languages like JavaScript which are synchronous by default and will proceed to the next line immediately a given line is interpreted.

In this article we looked at ways to prevent this apparent loss of data by handling asynchronous operations. The methods discussed include:
1. **Callbacks**: a function passed into another function.
2. **Promises**: an javascript object that tracks the status of an asynchronous task.
3. **Async and await**: a modern way to make asynchronous code look synchronous.

<br>

Please leave a like if you found value in this article and share to help others. If you have any issues following the instructions in the article or disagree with anything in it, kindly let me know in the comments section. 

Till next time friends.
