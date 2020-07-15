# react-reference

Reference of the material presented in Tyler McGinnis' course on react, a 
JavaScript library for building user interfaces.

## Updating state

State is updated using `this.setState`. There are two ways of using it.
the first way is to pass it an object which gets merged to the current state

```js
this.setState({
  name: newName
})
```

The other way is to pass a function that is passed the current state and
returns an object that is merged with the new state.

```js
this.setState((state) => {
  return {
    friends: state.friends.concat(newFriend)
  }
})
```

### Update method caveats

When making an update method there must be following line in the constructor

```js
this.updateName = this.updateName.bind(this)
```

The other option is to use the "Class Fields" proposal and an arrow function 

```js
handleChange = (event) => {
    this.setState({
      username: event.target.value
    })
  }
```

## Rendering Lists

Map the list and return `<li>` tags making sure to add a unique `key` prop to
each list item.

```jsx
<ul>
  {tweets.map((tweet) => (
    <li key={tweet.id}>
      {tweet.text}
    </li>
  ))}
</ul>
```

## Component Lifecycle

This is a list of common lifecycle methods, and what they're used for.
In React, your view is a function of your state. For this reason, lifecycle 
methods are only concerned with state and React takes care of the rendering.

### `constructor`

Used to set the initial state of the component

### `render`

* used to describes the type of DOM node you want to render. It must be a pure 
function so it can't have any side effects.
* invoked when the component is first added to the DOM and any time its state
changes. 


### `componentDidMount`

* used to make Ajax requests, and set up listeners
* invoked only one time when the component is first mounted to the DOM.

### `componentDidUpdate`

* used to re-fech data without having to remount the component
* invoked after the component's local state changes or after it receives new
props. It is not invoked on the initial render
* passed the component's previous props and the component's previous state
as arguments

### `componentWillUnmount`

* used for cleaning up and removing listeners
* invoked when the component is about to be removed from the DOM.

### Example

Here is an example for how to handle updating a component via props

```jsx
class Repos extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      repos: null
    }
  }
  componentDidMount() {
    fetchRepos(this.props.language)
      .then((repos) => {
        this.setState({ repos })
      })
  }
  componentDidUpdate(prevProps, prevState) {
    if (this.props.language !== prevProps.language) {
      this.setState({repos: null})

      fetchRepos(this.props.language)
        .then((repos) => {
          this.setState({ repos })
        })
    }
  }
  render() {
    if (this.state.repos === null) {
      return <Loading />
    }

    return <Grid data={this.state.repos} />
  }
}
```


## Controlled vs Uncontrolled Components

This terminology is common when dealing with forms.  
In a **controlled** component the form state lives inside of the component's 
state. 
In an **uncontrolled** component you don't have any component state, the 
form state lives inside the DOM

Using a text input as an example of a controlled component, setting and
accessing the input field is done through the components state.

```jsx
class Form extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      email: ''
    }

    this.handleChange = this.handleChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }
  handleChange(e) {
    this.setState({
      email: e.target.value
    })
  }
  handleSubmit() {
    alert('The email is ' + this.state.email)
  }
  render() {
    return (
      <div>
        <pre>The email is {this.state.email}</pre>
        <br />
        <input
          type='text'
          placeholder='Email'
          value={this.state.email}
          onChange={this.handleChange}
        />
        <button onClick={this.handleSubmit}>Submit</button>
      </div>
    )
  }
}
```

## Children in React

Whatever is between the opening and closing tag of an element will be 
accessible inside of the component via `props.children`

```jsx
function Header({ children }) {
  <h1>
    {children}
  </h1>
}

<Header>Title!</Header>
```

## Default Props

### Class Component

Add a static property of `defaultProps`

```jsx
StarRating.defaultProps = {
  color: '#ECB244'
}
```

### Function Component

Use ES6 default parameters

```jsx
function StarRating({ color = '#ECB244' }) {
  ...
}
```

## Higher Order Components

A higher order componenent:
* Is a component
* Takes in a component as an argument
* Returns a new component
* The component it returns can render the original component that was passed in

```jsx
function higherOrderComponent (Component) {
  return class extends React.Component {
    render() {
      return <Component />
    }
  }
}
```

### Example

The following higher order component adds hovering state to the passed 
component and allows the passed propname to be renamed

```jsx
function withHover(Component, propName = 'hovering') {
  return class WithHover extends React.Component {
    state = { hovering: false }
    mouseOver = () => this.setState({ hovering: true })
    mouseOut = () => this.setState({ hovering: false })
    render() {
      const props = {
        [propName]: this.state.hovering,
        ...this.props // pass through original props
      }

      return (
        <div onMouseOver={this.mouseOver} onMouseOut={this.mouseOut}>
          <Component {...props} />
        </div>
      );
    }
  }
}
```

## Render Prop

Allows functionality to be added to a component by wrapping it in the `render `
prop of another component. The `render ` prop is a function that the wrapper 
component uses to pass information to the wrapped component.

```jsx
class WrapperComponent extends React.Component {
  // state and state-altering functions ...
  
  render() {
    return (
      <div>
        {this.props.render(this.state.data)}
      </div>
    )
  }
}

function App() {
  return (
    <WrapperComponent render={(data) => 
      <WrappedComponent data={data} />
    }/>
  )
}
```

The `children` prop can be used in place of a render prop so that wrapping
your component will look like this:

```jsx
function App() {
  return (
    <WrapperComponent>
      {(data) => <WrappedComponent data={data} />}
    </WrapperComponent>
  )
}
```

### Example

```jsx
// wrapper component
class Hover extends React.Component {
  state = { hovering: false }
  mouseOver = () => this.setState({ hovering: true })
  mouseOut = () => this.setState({ hovering: false })
  render() {
    return (
      <div onMouseOver={this.mouseOver} onMouseOut={this.mouseOut}>
        {this.props.render(this.state.hovering)}
      </div>
    )
  }
}

// component to be wrapped
function TrendChart (props) {
  return (
    <>
      {props.hovering === true
        ? <Tooltip id='trend' />
        : null}
      <Chart type='trend' />
    </>
  )
}

function App() {
  return (
  <>
    <Hover render={(hovering) =>
      <TrendChart hovering={hovering} />
    }/>
  </>
}
```

## Contexts

Create a context with `React.createContext()` which gives two properties, 
`Provider` and `Consumer`, which are components.

`Provider` allows us to declare the data that we want available throughout 
our component tree

`Consumer` allows any component in the component tree that needs that data 
to be able to subscribe to it

### Using Provider

pass it a `value` prop. This will be the data that will be available to any
of its children that need to consume it

```js
<MyContext.Provider value={data}>
  <App />
</MyContext.Provider>
```

### Using Consumer

to access the data passed as a `value` prop to `Provider`, use a render prop

```js
<MyContext.Consumer>
  {(data) => {
      ...
    )
  }}
</MyContext.Consumer>
```

### Updating Context State

The following example demonstrates how to do this. Now a `Consumer` can 
pass `toggleLocale` to something like a button.

```jsx
class App extends React.Component { 
  constructor(props) {
    super(props)

    this.state = {
      locale: 'en',
      toggleLocale: () => {
        this.setState(({ locale }) => ({
          locale: locale === "en" ? "es" : "en"
        }));
      }
    }
  }
  render() {
    return (
      <LocaleContext.Provider value={this.state}>
        <Home />
      </LocaleContext.Provider>
    )
  }
}
```

## React Router

Import `BrowswerRouter` (often as `Router`), `Route`, and `Link` from 
`'react-router-dom'`

When React Router renders a component, it passes that component three things:
`match`, `location`, and `history`

To make React Router work `Router` must be rendered at the route of the application

To render a component based on the apps path use `Route`

```jsx
<Router>
  <div>
    <Route path='/' component={Home} />
  </div>
</Router>
```

Use the `exact` prop to avoid rendering multiple components

```jsx
<Router>
  <div>
    <Route exact path='/' component={Home} />
    <Route path='/about' component={About} />
    <Route path='/topics' component={Topics} />
  </div>
  </Router>
```

To change the app's location use the `Link` component

```jsx
<Link to='/'>Home</Link>
```

When nextring routes, use `props.match.url` to dynamically build the path

```jsx
<Link to={`${match.url}/rendering`}>
```

### Query Strings

A query string is accessed with `props.location.search`

the string can be parsed with the 
[query-string](https://www.npmjs.com/package/query-string) library on NPM

To set a query string

```jsx
<Link
  to={{
    pathname: '/battle/results',
    search: `?playerOne=${playerOne}&playerTwo=${playerTwo}`
  }}
>
```

### 404 Page

use the `<Switch>` where the last child route has no path to render a 404
page

```jsx
<Switch>
  <Route path="/" exact component={Home}/>
  <Route path="/will-match" component={WillMatch}/>
  <Route component={NoMatch} />
</Switch>
```

## Code Splitting

```jsx
const Analytics = React.lazy(() => import('./Analytics'))
const Settings = React.lazy(() => import('./Settings'))

function App () {
  return (
    <div>
      <React.Suspense fallback={<Loading />}>
        <Analytics />
        <Settings />
      </React.Suspense>
    </div>
  )
}
```


## Class Fields
class field proposal, how to use it with webpack/babel
TODO: understand lexical binding of `this` in arrow functions
performance consideratoins with class fields

## Hosting With Netflify

