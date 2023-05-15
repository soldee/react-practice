# React

- React is a Javascript library for building user interfaces
- React DOM is the interface to the web (el que connecta els elements de React amb el DOM real)

<br>

### How does React work ?

- The virtual DOM (VDOM) is a programming concept where a “virtual”, representation of a UI is kept in memory and synced with the “real” DOM by a library such as ReactDOM. This process is called reconciliation.
- This approach enables the declarative API of React: 
    - You tell React what state you want the UI to be in, and it makes sure the DOM matches that state. 
    - This abstracts out the attribute manipulation, event handling, and manual DOM updating that you would otherwise have to use to build your app.

- React only cares about **props**, **context** and **state**
    - Whenever this elements are modified, Components that use this elements are re-evaluated

- **Re-evaluating** vs **re-rendering**
    - Re-evaluating Components: 
        - Components are re-evaluated whenever state,props or context changes
        - React re-executes component functions
        - Re-evaluating a component means that all its child components will also be re-evaluated
    - Re-rendering the DOM
        - Changes to the real DOM are only made for differences between evaluations
        - *The real DOM is only modified when needed.* 
            - This is important for performance 
            - Making a virtual comparison between the previous state and the current state is fairly cheap
            - Changing things on the real DOM is more expensive.
        - *The real DOM is only modified where needed.* 
            - Ex: Si re-evaluem tots els components pero només canvia una \<h2> de un dels components, React DOM s’encarregara de modificar només aquest \<h2> per millorar performance

- React.memo()
    - Re-evaluating a component means that all its child components will also be re-evaluated
    - With React.memo() we can tell React to compare the previous and the current props, and to only re-evaluate the component if they are different.
    - Why not apply React.memo() to all Components ?
        - React has to compare the values which in turn has some overhead
        - En general, si tenim un component tree molt gran ens podria ser útil aplicar-ho al parent si sabem que aquest no s’ha de reevaluar sempre 
    - Per darrere React.memo() utilitza el ‘===’ operator.
        - false === false  		true
        - [1,2]===[1,2]		false
        - ‘hi’===’hi’		true

        - Això vol dir que la comparació de funcions donarà sempre false si no es tracta de la mateixa funció en memoria. Podem evitar aquest problema utilitzant useCallback hook.

<br>

- useCallback hook
Ex: 
``` js
const show = useCallback(() => {
    setShow((prev) => {
        if (buttonsToggled) !prev
    });
)}, [buttonsToggled]);
```
    - Fem que es faci un ‘cache’ de la funció i s’anirà a buscar aquesta mateixa sempre que es re-evalui el component que la conté
    - També admet un array de dependències opcional que serveix per a regenerar la funció si algun valor de les dependències de la funció canvia

<br>

- Components and state
    - React manages variable state using useState and useReducer.
    - Variables managed by these hooks are initialized once by react with the default value we pass. 
        - Podria ser que algun component s’elimini del reactDOM (per exemple si renderitzem components condicionalment); en aquest cas, si es torna a renderitzar el component les variables de useState/useReducer es reinicialitzen al valor indicat al hook

<br>

- State updates and scheduling
    - When we use a setState function, React schedules a state update with the data passed to the function. 
    - That is because React is performing other potential performing tasks at the moment.
    - However, React guarantees that when calling the same setState function, these are processed in the order they were scheduled.
    - Because of the fact that there might be multiple state changes scheduled, it is recommended to use ‘setState((prevState)=> …)’ when updating based on the previous state.

    - Ex: 
        - tenim const [bool,setBool] = useState(true)
        - tenim setBool(!bool)
        - imaginem que es crida la funcio setBool dues vegades pero React no les executa immediatament sino que fa el scheduling de les funcions.
        - Aleshores tindrem les següents funcions pendents d’executar:
            - setBool(false)
            - setBool(false)
        - El segon setBool pren el mateix valor perque la primera funcio encar no ha modificat l’estat a false.
        - Per a evitar-ho, si utilitzem una callback ‘setBool((prevBool)=>!prevBool)’ aleshores el valor de prevBool no es passa a la funció fins que aquesta s’ha d’executar de manera que ens assegurem que tingui l’últim estat.  

<br>

- Batching
    - All state updates made in the same code block are batched together by React and updated together.
    - És a dir, si tenim dos o més setState() dins d’una funció (sense callbacks ni promises entre mig), React junta els dos updates i els fa simultàneament de manera síncrona.

<br>

- useMemo hook
    - hook for memoing (caching) the result of a calculation between re-renders
    - const sortedList = useMemo(() => { return items.sort((a,b) => a-b) }, [items]);
        - fem que la sortedList només es recalculi entre re-renders si es modifica la variable items
        - això podria millorar la performance de la aplicació ja que fer un sort es bastant memory-intensive


### Class-based components
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

### Sending HTTP requests


