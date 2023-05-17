## React Router

### Dynamic routes
- Podem utitlitzar paths que poden canviar depenent per exemple de un productId (com en aquest exemple)
- Per a fer-ho utilitzem un placeholder a la creació del router utilitzant ":" i podem accedir al valor del placeholder amb el useParams hook i el nom especificat al placeholder

```js
function ProductDetail = () => {

    const params = useParams();
    return (<h1>Product {params.productId} details</h2>);
}

function App = () => {
    const router = createBrowserRouter([
        {
            path: '/',
            element: <RootLayout />,
            errorElements: <Error />,
            children: [
                { path:'/', element: <Home /> },
                { path:'/products/:productId', element: <ProductDetail /> }
            ]
        }
    ])
}
```
<br>

### Relative vs Absolute paths
- Podem passar un prop "to" al Link i opcionalment un prop "relative" que per defecte pren el valor de "route". Això vol dir que és un path relatiu a la ruta i, per tant, si cliquem el link "Back" ens portaria a /root
- Si volem que sigui relatiu al path hem d'especificar-ho; Si cliquem el Link  "Back relative to path" pujarem un nivell
```js
function App = () => {
    const router = createBrowserRouter([
        {
            path: '/api',
            element: <RootLayout />,
            errorElements: <ErrorPage />,
            children: [
                { path:'products', element: <HomePage /> }, // relative path; resolves to /api/products
                { path:'/api/products/:productId', element: <ProductDetail /> } // absolute path
            ]
        }
    ])
}

function ProductDetail = () => {

    const params = useParams();
    return (
        <>
            <h1>Product {params.productId} details</h2>
            <Link to="..">Back</Link>  // relative path to go up one level relative to the current route
            <Link to=".." relative='path'>Back relative to path</Link>  // relative path to go to up one level relative to the current path
        </>
    );
}
```
<br>

- We can use the special prop "index" on a path which simply means it should render if the parent path is active.
```js
children: [
    { path:'products', element: <HomePage /> }
]
```
same as:
```js
children: [
    { index: true, element: <HomePage> }
]
```