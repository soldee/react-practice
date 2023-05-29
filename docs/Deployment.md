## Deployment

Deployment of react apps follows some steps:
1. Test code
2. Optimize code
3. Production build

<br><br>

### Optimizing code

- Lazy loading
    - We can add lazy loading to our react app to optimize it and to only load components when really needed.
    - Aixo farà que el codi js del component només es descarregui quan s'ha de renderitzar
    - To do so, we firstly need to remove the imports of the component we want to load lazily and import it by passing a function:

```js
const BlogPage = lazy() => {
    import('./pages/Blog');
}

//router
{ 
    index: true, 
    element: <Suspense fallback={<p>Loading...</p>}><BlogPage /></Suspense>, 
    loader: () => import('./pages/Blog').then(module => module.loader()) 
}
```

<br><br>

### Production build
```sh
npm run build
```
This will create an optimized production build
- It creates a build folder with all the frontend files
- In the js folder we have all the javascript code.
    - The main js file contains all the code (including all the dependencies and third-party libraries) and it is the file that is initially loaded 
    - The chunk files reference lazyly loaded classes that will be downloaded from the backend when needed

<br><br>

Note that when deploying react apps we have to take into account that React uses the SPA (Single Page Application) principle. 

Hauriem de configurar el nostre server per a que sempre retorni index.html i els mateixos fitxers js independentment de la ruta especificada a la url (ja que aquesta està controlada per React).