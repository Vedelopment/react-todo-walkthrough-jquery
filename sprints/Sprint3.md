## Sprint 3: Fetching Data with Axios

React actually isn't as full-featured as some front-end frameworks like AngularJS or BackboneJS.  

React relies on third party libraries to fetch data.

Today, we'll be using a library called [Axios](https://github.com/mzabriskie/axios), a promise-based HTTP client for the browser and node.

1. Install `axios` and create the directory and file that will manage AJAX requests:

```bash
$ npm install axios --save
$ mkdir src/models
$ touch src/models/Todo.js
```

2. We are going to the super-crud API endpoints for `todos` to get some data. Take a look at the raw json at https://super-crud.herokuapp.com/todos.


3. In `src/models/Todo.js`, add the following code:

```js
import axios from 'axios'

class TodoModel {
  static all(){
    let request = axios.get("https://super-crud.herokuapp.com/todos")
    return request
  }
}

export default TodoModel
```


3. Think critically about the steps in this sprint so far. Are you creating a new database model? Or is this a client-side "model" as in "MVC" or "MVVM"?

4. Think critically about the `TodoModel` class.  What is a "static" method in a JavaScript class?

  > In the `TodoModel` class, `all()` is a static method. What does this mean? A static method can be called without an **instance** of the class. This will allow us to call `all()` in the following way (without ***instantiating*** the class with new): `let todos = TodoModel.all()`. [More on Static Methods in JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes#Static_methods)


5. Think critically about the `TodoModel.all` static method.  What is `axios`? What do you think it is doing in the last code snippet?

  > Axios is a tool for making AJAX requests, and its API is pretty intuitive! When we use the `all` method on our `TodoModel`, it will make a `GET` request to the super-crud url for all todos. We tell `all` to return the request so that we can chain promises from it. (The promises will tell our code what happens next.)


5. This file isn't connected to anything yet, o 
We can't really test out the code in this file in isolation, so we must `import` it into our application in order to test it. The logical place to import this code is in the `TodosContainer` component.

For now, let's toss this in the `TodosContainer`'s `render()` method: this isn't ultimately going to be where we want to call `TodoModel.all()`, but for testing purposes, it will suffice.

In `components/TodosContainer.js`:

```js
import React, {Component} from 'react'
import TodoModel from '../models/Todo'

class TodosContainer extends Component {
  render(){
    TodoModel.all().then( (res) => {
      console.log(res);
    })
    return (
      <div className='todosContainer'>
        <h2>This is a todos container</h2>
      </div>
    )
  }
}

export default TodosContainer
```

Awesome, we can see the response from our database as soon as the page loads, we know it's working! However, its completely in the wrong spot and we don't have anything we're passing todos to... yet!

Now that we can get our data, let's code how we present that data. It'll be a bit before we connect these pieces and actually see our todos in our app, but just hold on we'll get there!

### Rendering A Todo
Let's start at the bottom and bubble up. It'll be nice if each todo we're its own component. To follow FIRST(Focused Independent Reusable Small Testable). Let's create `src/components/Todo.js` and put the following in it:

```js
import React, {Component} from 'react'

class Todo extends Component {
  render(){
    return(
      <p data-todos-index={this.props.todo.id}>
        <span>{this.props.todo.body}</span>
      </p>
    )
  }
}

export default Todo
```

When we write this component we know that if we pass it a `todo`, as a `prop`, that has both an id and a body, that it will render. AND it will render the same way every time. So what will be rendering each individual `Todo` component?

### Rendering Todos
We need another component. Its responsibility will be to render all of the todos. Let's create another component `src/components/Todos.js` and fill it with the following:

```js
import React, {Component} from 'react'
import Todo from './Todo'

class Todos extends Component {
  render(){
    let todos = this.props.todos.map( (todo) => {
      return (
        <Todo
          key={todo._id}
          todo={todo}/>
      )
    })
    return(
      <div className="todos">
        {todos}
      </div>
    )
  }
}

export default Todos
```

In this component, we have a property called todos. When we eventually use this component, we need to pass it that property. Once we have our todos, it takes each one and maps a `Todo` component to the variable `todos`. Then renders all of the todos. We can use the map function to render multiple components for each individual todo and store them in a variable. We just need to make sure we bind `this` in case we need to access properties from the `Todos` component later.

### Putting it all together, at last! Todos

Let's shove the remaining code we need in and then let's talk about it. In `src/containers/TodosContainer.js`:

```js
import React, {Component} from 'react'
import TodoModel from '../models/Todo'
import Todos from '../components/Todos'

class TodosContainer extends Component {
  constructor(){
    super()
    this.state = {
      todos: []
    }
  }
  componentDidMount(){
    this.fetchData()
  }
  fetchData(){
    TodoModel.all().then( (res) => {
      this.setState ({
        todos: res.data.todo,
        todo: ''
      })
    })
  }
  render(){
    return (
      <div className="todosComponent">
        <Todos
          todos={this.state.todos} />
      </div>
    )
  }
}

export default TodosContainer
```

If we take a look at our browser now... BAM todos! What just happened....

```js
constructor(){
  super()
  this.state = {
    todos: []
  }
}
```

This is just like initialize in ruby(only a bit different). `constructor()` is basically a function that invokes when an instance of our class gets initialized. When we call `super()` were basically saying invoke the same `constructor` function that the React library defines for their `constructor`. In addition to that initialize a state for this component in which todos is a property and set its value as an empty array. We can then set the state any other time in our application using `.setState()`.

```js
fetchData(){
  TodoModel.all().then( (res) => {
    this.setState ({
      todos: res.data,
      todo: ''
    })
  })
}
```

This function leverages our model to retrieve our todos from our backend. In the promise of that request we set the state of this container component to have todos be the value returned from the response. Any time `setState` is invoked the component re-renders.

```js
componentDidMount(){
  this.fetchData()
}
```

### Hooks
Every component in react undergoes a component lifecycle. There are several "hooks" throughout this lifecycle. You can think of hooks like events that we can trigger functionality on. `componentDidMount` is a reserved hook that happens after a component renders. There are many hooks, this is a [great blog post](http://busypeoples.github.io/post/react-component-lifecycle/) that goes into much better detail of the lifecycle of a component.


You might be asking yourself: "Wait, why are we getting the data after the components already been rendered?" ([Andy did too](http://stackoverflow.com/questions/39338464/reactjs-why-is-the-convention-to-fetch-data-on-componentdidmount))

That's because a re-render will always happen because fetching data happens asynchronously. Here's the [Facebook recommendation](https://facebook.github.io/react/tips/initial-ajax.html)

### Passing State from parents to children
How have we passed state? What do we mean by state with reference to a react component? The state of the `TodosContainer` is simple, the todos. How does each individual todo know about the todo they need to render? From the state of the most parent container, `TodosContainer`

If we take a look at the `props` being passed from one component to the next, we can clearly see the chain of how information was passed.

In `src/containers/TodosContainer.js`:


```javascript  
<Todos
  todos={this.state.todos} />
```

In `src/components/Todos.js`:  

```js
  let todos = this.props.todos.map( (todo) => {
  return (
    <Todo
      key={todo.id}
      todo={todo}
    />
  )
})
```

In `src/components/Todo.js`:

```js
<p data-todos-index={this.props.todo._id}>
  <span>{this.props.todo.body}</span>
</p>
```

### PAUSE - Why is this awesome?
We could stop the lesson here and take this knowledge and build lots of cool things with it. Most of the API's developers have access to are read-only. That said, if we know an endpoint to get data, we now know how to use React to display that data.