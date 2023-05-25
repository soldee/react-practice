## React Router

- Different URL paths load different content on the screen
- Building SPAs (Single Page Application)
  - Only one initial HTML request & response
  - Page (URL) changes are then handled by client-side code
    - Changes the visible content without fetching a new HTML file

- Installation
  - npm install react-router-dom

<br>

### Minimal setup
```js
const router = createBrowserRouter([
  { path: '/', element: <HomePage /> },
  { path: '/products', element: <Products /> }
]);

function App() {
  return <RouterProvider router={router} />
}
```

- Browsing between pages
  - If we change the page like this, it will cause the browser to send an HTTP request to the backend to reload all the js files.
```js
function HomePage() {
	return <>
	  <h1>My home page</h1>
	  <p>Go to <a href="/products">Products</a></p>
	</>
}
```
Instead, we should use Link:
```js
function HomePage() {
  return <>
    <h1>My home page</h1>
    <p>Go to <Link to="/products">Products</Link></p>
  </>
```

<br>

### Layouts and nested routes
  - We can use this to wrap routes.
  - In this example we wrap all routes with a RootLayout that will always render a MainNavigation component and then will render the children depending on the current path

```js
const router = createBrowserRouter([
  { 
    path:'/', 
    element:  <RootLayout />,
    children: [
      { path: '/', element: <HomePage /> },
      { path: '/products', element: <Products /> }
    ]
  }
]);

function MainNavigation() {
    return (
        <header>
            <nav>
                <ul>
                    <li><Link to="/">Home</Link></li>
                    <li><Link to="/products">Products</Link></li>
                </ul>
            </nav>
        </header>
    );
}

function RootLayout() {
    return(
        <>
            <MainNavigation />
            <Outlet />
        </>
    );
};
```

<br>

### Showing error pages
- When we try to navigate to a page that is not mapped to any url, react-router throws a default error.
- We can modify this error by using errorElement prop to render the component that we need:
```js
const router = createBrowserRouter([
  { 
    path:'/', 
    element:  <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      { path: '/', element: <HomePage /> },
      { path: '/products', element: <Products /> }
    ]
  }
]);
```

<br>

### NavLink
- Like a react-router link but accepts a new prop called className which in turn accepts a function to which we can pass a isActive property.
- isActive property is a boolean that indicates controlled by react-router that indicates wheter the link is active or not
- we can use this to control the CSS class of the Link to for example highlight the Products link when we are in the products page

```js
function MainNavigation() {
  return (
    <header>
    <nav>
      <ul>
        <li><NavLink 
            className={({isActive}) => (isActive ? classes.active : undefined) }  
            to="/"
        >
        Home
        </NavLink></li>
        <li><NavLink 
          className={({isActive}) => (isActive ? classes.active : undefined) } 
          to="/products"
        >
        Products
        </NavLink></li>
      </ul>
    </nav>
    </header>
  );
}
```

<br>

### Routing programmatically
- We can use useNavigate hook to navigate programatically between pages
```js
const navigate = useNavigate();
  const navigateHandler = () => {
    navigate('/products');
  };

  return <>
    <h1>My home page</h1>
      <p>Go to <Link to="/products">Products</Link></p>
      <button onClick={navigateHandler}>Navigate</button>
  </>
```


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

<br>

### Loader function
- A function that we can pass to any route that will get executed by React before a specific route gets rendered
- If we use an async function inside the loader, this types of functions return a Promise, but React will check if the returned object is a promise and get the object for us.
- We can use the object returned by the loader function by using the useLoaderData hook
  - The returned data can be used only by the rendered Page or subcomponents inside it
- Note that to keep our App.js file leaner, we could declare the loader function inside the EventsPage.js file

```js
const router = createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      children: [
        { index: true, element: <HomePage /> },
        { 
          path: 'events', 
          element: <EventsRootLayout />,
          children: [
            { 
              index: true, 
              element: <EventsPage />,
              loader: async () => {
                const response = await fetch('http://localhost:8080/events');

                if (response.ok) {
                  //...
                } else {
                  const resData = await response.json();
                  return resData.events;
                }
              }
            },
            { path: ':event_id', element: <EventDetailPage /> },
            { path: 'new', element: <NewEventPage /> },
            { path: ':event_id', element: <EditEventPage /> }
          ]
        }
      ]
    }
]);

const EventsPage = () => {
  const events = useLoaderData();
  
  return (<EventsList events={events} />);
}
```

<br>

### loaders
- Accessing loader data from other routes
```js
// we access the loader data from a nested children using this hook
function EventDetailPage() {
    const data = useRouterData('event-detail');
    ...
}

// router
{
    path: ':eventId',
    id: 'event-detail',
    loader: eventDetailLoader,
    children: [
        { index: true, element: <EventDetailPage /> },
        { path:'edit', element: <EditEventPage /> }
    ]
}
```

<br>

### Handling errors
- We can include an errorElement in our router to handle all children errors

```js
const router = createBrowserRouter([
    {
      path: '/',
      element: <RootLayout />,
      errorElement: <ErrorPage />,
      children: [
        { index: true, element: <HomePage /> },
        { 
          path: 'events', 
          element: <EventsRootLayout />,
          children: [
            { 
              index: true, 
              element: <EventsPage />,
              loader: eventsLoader
            },
            { path: ':event_id', element: <EventDetailPage /> },
            { path: 'new', element: <NewEventPage /> },
            { path: ':event_id', element: <EditEventPage /> }
          ]
        }
      ]
    }
  ]);

const ErrorPage = () => {

    const error = useRouteError();

    let title = 'An error occurred!';
    let message = 'Something went wrong';

    if (error.status === 500) {
        message = error.data.message;
    }
    if (error.status === 404) {
        title = 'Not found!';
        message = 'Could not find resource or page';
    }

    return (
        <>
            <MainNavigation />
            <h1>{title}</h1>
            <h3>{message}</h3>
        </>
    );
}
```
Then we can throw errors from a loader() function (for example), that are going to be catched by the Error component defined in the router
```js
export async function loader() {
    const response = await fetch('http://localhost:8080/events');

    if (response.ok) {
      throw new Response(JSON.stringify({message: 'Could not fetch events'}), {status: 500})
    } else {
      const resData = await response.json();
      return resData.events;
    }
}
```
We use the Response object to throw errors so that we can include the status property


<br>

### actions
- To send data using React Router we can use actions
- **React Router provides a Form component that triggers an action when we submit the form**
- The action function receives an object {request, params} with the data from the form
  - We can access values from the form using request.formData().get()
```js
function EventForm() {
    <Form method='post'>
        <div>
            <label>Title</label>
            <input name='title'/>
        </div>
        <div>
            <label>Date</label>
            <input name='date'/>
        </div>
    </Form>
}

export default EventForm;
```
```js
function NewEventPage() {
    return <EventForm />;
}

export default NewEventPage;

export async function action({request, params}) {
    const data = await request.formData();

    const eventData = {
        title: data.get('title'),
        date: data.get('date')
    }

    fetch('http://localhost:8080/events', {
        method: 'POST',
        headers: { 'Content-Type: application/json'}
        body: JSON.stringify(eventData)
    });

    ...
    return redirect('/events');
}

//router
import NewEventPage, {action as newEventAction} from '...';
{ path:'new', element: <NewEventPage />, action: newEventAction }
```
- We also use the redirect function to redirect the user to the desired page upon form submission

<br>

- **useSubmit**
  - We can also trigger actions programmatically using the useSubmit hook
  - When we call the submit action the action function gets triggered
```js
function EventItem() {
    const submit = useSubmit();

    function startDeleteHandler() {
        const proceed = window.confirm('Are you sure ?');

        if (proceed) submit(null, { 'method': 'delete' })
    }
    ...
}
```
```js
function EventDetailPage() {
    ...
    return <EventItem />
}
export async function action({params, request}) {
    ...
}

//router 
import EventDetailPage, { action as deleteEventAction } from '...';
{ element:<EventDetailPage />, action: deleteEventAction }
```

<br>

- **useNavigation**
  - Hook to get state of the current transition from one route to another (f.e. when we submit a Form)
  - In this example we could use this hook to update UI based on submission status
    - we use the isSubmitting to conditionally disable the button if the form is currently submitting
```js
function EventForm({method, event}) {
    const navigate = useNavigation();
    const isSubmitting = navigation.state === 'submitting';

    <Form>
        ...
        <button type='submit' disabled={isSubmitting}>
            { isSubmittin ? 'Submitting...' : 'Submit' }
        </button>
    </Form>
    ...
}
```

<br>

- **useActionData**
  - Hook to get the data returned from an action
  - Normalment ho utilitzem per tractar la resposta que rebem del backend al fer una action
  - Ex:
    - tenim un backend que retorna aixo:
```js
router.post('/', async (req, res, next) => {
    const data = req.body;

    let errors = {};
    if (!isValidText(data.title)) errors.title = 'Invalid title.';
    if (!isValidText(data.description)) errors.description = 'Invalid description.';

    if (Object.keys(errors).length > 0) {
        return res.status(422).json({
        message: 'Adding the event failed due to validation errors.', 
        errors
        });
    }
    res.status(200);
});
```
<br>

- Podem utilitzar el que retorna una accio que fa un post a aquest backend per a mostrar els errors.

```js
const EventForm() {
    const data = useActionData();

    return(
        {data && data.errors && Object.values(data.errors).map((err) => {
            <li key={err}>{err}</li>
        })}
    );
}
``` 

<br>


- **useFetcher**
  - Hook that we should use to interact with some action or loader without triggering any route changes
  - Retorna dos valors
    - data: to get the data returned from an action or loader
    - state: *idle || loading || submitting* . Get information about the state of the action that was triggered.
  - In this example 
    - we have a NewsLetterSignup component that is always shown in the main navigation object. Per tant, si estem en una ruta qualsevol i ens subscribim a un Newsletter no volem transicionar a la ruta de Newsletter sino que ens volem quedar a la ruta actual.
    
```js
function NewsletterSignup() {
  const fetcher = useFetcher();
  const { data, state } = fetcher;

  useEffect(() => {
    if (state === 'idle' && data && data.message) {
      window.alert(data.message);
    }
  }, [data, state]);

  return (
    <fetcher.Form
      method="post"
      action="/newsletter"
      className={classes.newsletter}
    >
      <input
        type="email"
        placeholder="Sign up for newsletter..."
        aria-label="Sign up for newsletter"
      />
      <button>Sign up</button>
    </fetcher.Form>
  );
}
```

<br>

- **Deferring data fetching**
  - defer function allows you to defer values returned from loaders by passing promises instead of resolved values
    - Això ens permet precarregar routes en les quals encara no s'ha rebut tota la informació a renderitzar però que anirem renderitzant conforme arribi la info de un backend per exemple.
  - En aquest exemple 
    - la variable events serà una promise. 
    - amb el component Await fem que es renderitzin els events una vegada es resol la promise
    - amb el component Suspense podem afegir un fallback a renderitzar mentre la promise events no s'ha resolt
```js
function EventsPage() {
  const { events } = useLoaderData();

  return (
    <Suspense fallback={<p style={{ textAlign: 'center' }}>Loading...</p>}>
      <Await resolve={events}>
        {(loadedEvents) => <EventsList events={loadedEvents} />}
      </Await>
    </Suspense>
  );
}

export default EventsPage;

async function loadEvents() {
  const response = await fetch('http://localhost:8080/events');

  if (!response.ok) {
    throw json(
      { message: 'Could not fetch events.' },
      {
        status: 500,
      }
    );
  } else {
    const resData = await response.json();
    return resData.events;
  }
}

export function loader() {
  return defer({
    events: loadEvents(),
  });
}
```
<br>

  - Això pot ser molt útil en pagines on hem de renderitzar molts components que depenen de la resposta de un backend que retornarà data amb diferents temps de resposta.
  - P.D també li podem passar un await a defer de la següent manera:
    ```js
    export function loader() {
        return defer({
            events: loadEvents(),
            mainEvent: await loadEvent(id)
        });
    }
    ```
    - D'aquesta manera esperarem a que carregui el main event (que és una request rapida) per anar a la nova route. Un cop a la nova route on es mostra el main event, es renderitzarà la resta de events quan retorni el backend


