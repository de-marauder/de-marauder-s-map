# React State and Props.

# Introduction
React is a JavaScript framework which utilizes the concept of the virtual DOM (Document Object Model) to facilitate building reusable and modular web components. Two of its most important features include the concept of state management and props.

This article explains these two concepts (state and props) and provides use cases as well as differences between these concepts.

## Prerequisites
To follow along smoothly with this tutorial, you’ll need the following:
- Basic knowledge of Javascript and HTML
- NodeJs

# What is a react state?
React is a framework for writing reusable and isolated web components. To do this efficiently, the concept of state is employed. 

So, what is state?

States in react are mutable local variables which are attached to an instance of a web component. These variables are unlike typical variables in the sense that they allow the conditional rendering and re-rendering of aspects of a web component. 

React states are one of the reasons why the content of a particular web component can be changed without reloading the browser.

## How to use state
Now that we have an understanding of what React states are, let’s write some code.

First off, we’ll need to install and setup react on our system. For this we shall use vite. Vite is a tool that can be used to build up projects quickly and efficiently. It supports different frameworks and libraries like react and even vanilla Js. 

To setup react using vite run the following command,
```sh
npm create vite@latest
```
The `@latest` tells npm (node package manager) to use the latest version of vite. Select `react` as your framework and set the name of your project.

![1-react-setup install.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665668249523/khd1maM1j.png align="left")

Select what language you would like to use to build your project. We shall use Javascript for this tutorial.

![2-react-setup-select_language.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665668272190/0Gv4S56rn.png align="left")

Run the final commands
```sh
cd my-react-app
npm install
npm run dev
```
At this point, your project directory should have the following structure,

![4-vite-react folder structure.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665668407012/KbDxAmKY2.png align="left")

Now that that’s out of the way. Open your new project in a code editor of your choice and open the `App.jsx` file in the `src` folder.

We shall begin by defining a functional component.

Replace the current piece of code with the following:

```js
import './App.css'
 
function App() {
 
 return (
   <div className="App">
    
   </div>
 )
}
 
export default App

```

In this example, we shall implement a simple solution using the `useState` hook which as you must have guessed allows us to use state in functional components. Our code looks like this,

```js
import { useState } from 'react';
import './App.css'
 
function App() {
 
 const [text, setText] = useState("I am a state variable")
 const [isChanged, setChange] = useState(false)
 
 function changeText () {
   setChange(!change)
   if (!isChanged) setText("I have been changed"); else setText('I am a state variable again')
 }
 
 return (
   <div className="App">
 
     <p>{text}</p>
 
     <button onclick={changeText}>CLICK ME</button>
    
   </div>
 )
}
 
export default App

```
The first thing we do here is import the `useState` hook from `react`. Then we define two sets of state variables. The `useState` hook returns an array of 2 items, the state variable and a function which is used to update the state variable. In our example, the 2 parameters are retrieved by destructuring the returned array. Also notice that the `useState` hook accepts an argument which is used as the initial value for the state defined.

Our simple functional component just returns a simple piece of jsx comprising a `p` tag containing the current value of the state variable `text`. It also includes a button with an `onClick` event handler attached to it. 

The event handler simply toggles between two different texts for the `text` state variable using the `isChanged` state variable to keep track of when state updates occur and determining what value to set `text` to.

## Characteristics of State variables
As you can see, the state variable allows us to update the DOM dynamically by listening for events or setting timed executions using the likes of `setTimeout` and `setInterval`. Some notable characteristics of state variables include:
1. Mutability: They can be changed over the lifetime of the component.
2. Localized: These state variables are local to the components they were created in and can only exist as “state variables in them”.
3. State variables are just like regular variables and so can be passed as props.


# What are Props?
Simply put, a prop is a variable passed down into a component from its parent. It is typically used to provide child components with information from their parent’s context. So all variables or objects created in a parent become props once they are passed down to its child components.

For clarity, A child component is a component which is wrapped by another component.

Let us create a folder called `components` and in it a file named `parent.jsx`. This file should include the code below in it.

```js
import React from 'react'

export default function Parent(props) {

    return (
        <div>
            --- Parent begins ---
            <p>{props.state}</p>
            {props.children}
            --- Parent ends ---
        </div>
    )
}

```

Notice how this new component's definition receives the `props` argument? This is the first indicator that information is being passed into this component from a parent element. The `props.children` object is used to access all components which this parent component will wrap. We shall see this soon.

Let us also create another component `child.jsx` in the `components` folder and include the following,

```js
import React from 'react'

export default function Child(props) {
  return (
    <div>
      --- child begins ---
      <p>Props ==> {JSON.stringify(props)}</p>
      <p>
        I am the child
      </p>
      --- child ends ---
    </div>
  )
}

 
```

FInally, Let us now make use of our newly created components in our `App.jsx` component.

```js
import { useState } from 'react';
import './App.css'
import Child from './component/child';
import Parent from './component/parent';
 
function App() {
 
 const [state, setState] = useState("I am a state variable")
 const [isChanged, setChange] = useState(false)
 
 function changeText () {
   setChange(!ischanged)
   if (!isChanged) setState("I have been changed"); else setState('I am a state variable again')
 }
 
 return (
   <div className="App">
 
     <p>{state}</p>
 
     <button onclick={changeText}>CLICK ME</button>
     <br />
     <br />
     <hr />
     <br />
     <br />
     <Parent state={state}>
       <Child changed={isChanged} />
     </Parent>
    
   </div>
 )
}
 
export default App
 
```

In our updated App component you’ll notice how the Parent component wraps the Child component. This is allowed by passing the `props.children` object somewhere in the returned `jsx` of the Parent component. 

We can also see how state is passed from this example. By defining inline variables similar to how you would define element attributes in regular HTML. 

Now if we run our project using 
```sh
npm run dev
```
We can examine the full functionality of states and props. You’ll notice how the state variables passed into the Parent and Child components are indeed available in them. You should also see the props object printed to the screen as a string and view the various properties available on it.

![5-final result.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665669679192/5QqYSkXaE.png align="left")


# Differences Between State and Props

| State | Props |
| ----- | ----- |
| **Mutable**: Can be updated | **Immutable**: Cannot be changed |
| Can cause the component to re-render when updated | Cannot be changed so does not cause re-renders |
| Is created and only exist as “state” within the context of its component | Created by passing down variables from parent to child components. |


# Similarities between State and Props
State and props also possess certain similarities in that they are both variables and as such hold data pertaining to a component. They can both be passed downward to child components (props drilling). Neither state nor props are available from child element to parent element.


# Conclusion
In this article, we have studied the meaning and purpose of states and props in react. We have also gone through some example cases of how these react concepts could be used in functional components. We also learnt about some of their differences and similarities.

In short these are crucial react concepts and essential for any react developer to know.

Thanks for reading. 

Please like and share if you found this article useful.
