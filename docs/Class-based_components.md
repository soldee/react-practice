## Class-based components
```js
class Product extends Component {
render() {
	return <h2>A Product</h2>
}
}
```

- Traditionally (React < 16.8), you had to use Class-based components to manage state.
- React 16.8 introduced hooks for functional components
- Class-based components can’t use hooks

- componentDidCatch()
    - lifecycle method que només està disponible en Class-based components
    - serveix per a fer un catch dels errors que llencin els childrens 

```js
class ErrorBoundary extends Component {
constructor() {
	super();
	this.state = { hasError: false };
}

componentDidCatch(error) {
	this.setState({hasError: true})
}
render() {
	if (this.state.hasError) {
		return <p>Something went wrong</p>;
}
	return this.props.children;
}
}
```

<br>