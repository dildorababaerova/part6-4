# part6-4
part 6-4 useReducer
We shall now use the React Query library to store and manage data retrieved from the server. 
Install the library with the command:
`npm install @tanstack/react-query`

```js
import React from 'react'
import ReactDOM from 'react-dom/client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

import App from './App'


const queryClient = new QueryClient()

ReactDOM.createRoot(document.getElementById('root')).render(

  <QueryClientProvider client={queryClient}>
    <App />

  </QueryClientProvider>
)
```
We can now retrieve the notes in the App component. The code expands as follows:

```js
import { useQuery } from '@tanstack/react-query'
import axios from 'axios'

const App = () => {
  // ...


  const result = useQuery({
    queryKey: ['notes'],
    queryFn: () => axios.get('http://localhost:3001/notes').then(res => res.data)
  })
  console.log(JSON.parse(JSON.stringify(result)))


  if ( result.isLoading ) {
    return <div>loading data...</div>
  }


  const notes = result.data

  return (
    // ...
  )
}
```

The first parameter of the function call is a `string notes` which acts as a `key` to the query defined, i.e. the list of notes.
The` return value of the useQuery function` is an `object` that indicates the status of the query. So the application retrieves data from the server and renders it on the screen without using the React hooks useState and useEffect. 
Let's move the function making the actual HTTP request to its own file requests.js:
```js
import axios from 'axios'

export const getNotes = () =>
  axios.get('http://localhost:3001/notes').then(res => res.data)
```
The App component is now slightly simplified
```js
import { useQuery } from '@tanstack/react-query' 

import { getNotes } from './requests'

const App = () => {
  // ...

  const result = useQuery({
    queryKey: ['notes'],

    queryFn: getNotes
  })

  // ...
}
```

### Synchronizing data to the server using React Query

Let's start by adding new notes.Let's make a function createNote to the file requests.js for saving new notes:
```js
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

export const getNotes = () =>
  axios.get(baseUrl).then(res => res.data)

export const createNote = newNote =>  axios.post(baseUrl, newNote).then(res => res.data)

export const updateNote = updatedNote =>
  axios.put(`${baseUrl}/${updatedNote.id}`, updatedNote).then(res => res.data)
```

The App component will change as follows:

```js
import { useQuery, useMutation } from '@tanstack/react-query'
import { getNotes, createNote } from './requests'

const App = () => {

 const newNoteMutation = useMutation({ mutationFn: createNote })

  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value
    event.target.note.value = ''

    newNoteMutation.mutate({ content, important: true })
  }

  // 

}
```

The change for the mutation adding a new note is as follows:

```js
const App = () => {
  const queryClient =  useQueryClient() 

  const newNoteMutation = useMutation({
    mutationFn: createNote,
    onSuccess: (newNote) => {

      const notes = queryClient.getQueryData(['notes'])
      queryClient.setQueryData(['notes'], notes.concat(newNote))
    }
  })
  // ...
}
```
### useReducer

We shall now implement the counter state management using a Redux-like state management mechanism provided 
by React's built-in useReducer hook. Code looks like the following:

```js
import { useReducer } from 'react'

const counterReducer = (state, action) => {
  switch (action.type) {
    case "INC":
        return state + 1
    case "DEC":
        return state - 1
    case "ZERO":
        return 0
    default:
        return state
  }
}

const App = () => {
  const [counter, counterDispatch] = useReducer(counterReducer, 0)

  return (
    <div>
      <div>{counter}</div>
      <div>
        <button onClick={() => counterDispatch({ type: "INC"})}>+</button>
        <button onClick={() => counterDispatch({ type: "DEC"})}>-</button>
        <button onClick={() => counterDispatch({ type: "ZERO"})}>0</button>
      </div>
    </div>
  )
}

export default App
```

The hook useReducer provides a mechanism to create a state for an application.

```js
const [counter, counterDispatch] = useReducer(counterReducer, 0)
```

The reducer function that handles state changes is similar to Redux's reducers, i.e. the function gets as 
`parameters` the `current state`(counter) and the `action`(counterDispatch) that `changes` the state.
`counterReducer` - reducer
`0` - initial state
Like Redux's reducers, actions can also contain arbitrary `data`, which is usually put in the action's `payload` field.

### Using context for passing the state to components

Let us now create a context in the application that `stores` the `state` management of the counter.
The context is created with React's hook createContext. Let's create a context in the file CounterContext.jsx:

```js
import { createContext } from 'react'

const CounterContext = createContext()

export default CounterContext
```

The App component can now provide a context to its child components as follows:

```js
import CounterContext from './CounterContext'
const App = () => {
  const [counter, counterDispatch] = useReducer(counterReducer, 0)

  return (
    <CounterContext.Provider value={[counter, counterDispatch]}>      <Display />
      <div>
        <Button type='INC' label='+' />
        <Button type='DEC' label='-' />
        <Button type='ZERO' label='0' />
      </div>
    </CounterContext.Provider>  )
}
```

Other components now access the context using the useContext hook:

```js
import { useContext } from 'react'import CounterContext from './CounterContext'

const Display = () => {
  const [counter] = useContext(CounterContext)  return <div>
    {counter}
  </div>
}

const Button = ({ type, label }) => {
  const [counter, dispatch] = useContext(CounterContext)  return (
    <button onClick={() => dispatch({ type })}>
      {label}
    </button>
  )
}
```
### Defining the counter context in a separate file

Now let's move everything related to the counter to CounterContext.jsx:

```js
import { createContext, useReducer } from 'react'

const counterReducer = (state, action) => {
  switch (action.type) {
    case "INC":
        return state + 1
    case "DEC":
        return state - 1
    case "ZERO":
        return 0
    default:
        return state
  }
}

const CounterContext = createContext()

export const CounterContextProvider = (props) => {
  const [counter, counterDispatch] = useReducer(counterReducer, 0)

  return (
    <CounterContext.Provider value={[counter, counterDispatch] }>
      {props.children}
    </CounterContext.Provider>
  )
}

export default CounterContext
```

The file now exports, in addition to the `CounterContext` object corresponding to the `context`, 
the `CounterContextProvider` component, which is practically
a context provider whose `value` is a `counter` and a `dispatcher` used for its state management.

Let's enable the context provider by making a change in main.jsx:

```js
import ReactDOM from 'react-dom/client'
import App from './App'
import { CounterContextProvider } from './CounterContext'
ReactDOM.createRoot(document.getElementById('root')).render(
  <CounterContextProvider>    
  <App />
  </CounterContextProvider>)
```


The App component is simplified to the following form:

```js
import Display from './components/Display'
import Button from './components/Button'

const App = () => {
  return (
    <div>
      <Display />
      <div>
        <Button type='INC' label='+' />
        <Button type='DEC' label='-' />
        <Button type='ZERO' label='0' />
      </div>
    </div>
  )
}

export default App
```


It is possible to make the code a bit more pleasant and expressive by defining a couple
of `helper functions` in the CounterContext file:

```js
import { createContext, useReducer, useContext } from 'react'
const CounterContext = createContext()

// ...

export const useCounterValue = () => {
  const counterAndDispatch = useContext(CounterContext)
  return counterAndDispatch[0]
}

export const useCounterDispatch = () => {
  const counterAndDispatch = useContext(CounterContext)
  return counterAndDispatch[1]
}
// ...
```

The Display component changes as follows:


```js
import { useCounterValue } from '../CounterContext'
const Display = () => {
  const counter = useCounterValue()  return <div>
    {counter}
  </div>
}


export default Display
```

Component Button becomes:

```js
import { useCounterDispatch } from '../CounterContext'
const Button = ({ type, label }) => {
  const dispatch = useCounterDispatch()  return (
    <button onClick={() => dispatch({ type })}>
      {label}
    </button>
  )
}

export default Button
```

As a technical detail, it should be noted that the helper functions useCounterValue and 
useCounterDispatch are defined as custom hooks, because calling the hook function useContext 
is possible only from React components or custom hooks. 
Custom hooks are JavaScript functions whose name must start with the word use.


