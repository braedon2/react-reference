# react-reference

Reference of the material presented in Tyler McGinnis' course on react

## Component Lifecycle

Here is a list of common lifecycle methods, and what they're used for.
In React, your view is a function of your state. For this reason, lifecycle 
methods are only concerned with state and React takes care of the rendering.

### `constructor`

Used to set the initial state of the component

### `render`

* used: to describes the type of DOM node you want to render. It must be a pure 
function so it can't have any side effects.
* invoked: when the component is first added to the DOM and any time its state
changes. 


### `componentDidMount`

* used: to make Ajax requests, and set up listeners
* invoked: only one time when the component is first mounted to the DOM.

### `componentDidUpdate`

* used: to re-fech data without having to remount the component
* invoked: after the component's local state changes or after it receives new
props. It is not invoked on the initial render
* passed: the component's previous props and the component's previous state
as arguments

### `componentWillUnmount`

* used: for cleaning up and removing listeners
* invoked: when the component is about to be removed from the DOM.

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

Using a text input as an example, setting and accessing the input field is done
through the components state.

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
