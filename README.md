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

We shall now implement the counter state management using a Redux-like state management mechanism provided by React's built-in useReducer hook. Code looks like the following:

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


