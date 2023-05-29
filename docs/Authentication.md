## Authentication

### Methods
- Server-side sessions
    - Server stores a session ID in a DB and sends it to the client when it is authenticated.
    - The client uses the session id for subsequent requests
- Token-based auth
    - Server generates a token and sends it to the client when it is authenticated
    - The client uses the token for subsequent requests and the server checks the token validity on each request

<br>

- La arquitectura de React fa que sigui adient per a token-based auth
- Es mostra un exemple de com consumir una API que proporciona un JWT al fer login i necessita autenticació per a la resta de peticions:
```js
{
    path: 'auth',
    element: <AuthenticationPage />
}

export async function action({request}) {
    const data = await request.formData();

    const authData = {
        email: data.getEmail('email'),
        password: data.get('password')
    };

    const response = fetch('http://localhost:8080/login', {
        method: 'POST',
        headers: {
            'Content-Type':'application/json'
        },
        body: JSON.stringify(authData)
    });

    if (!response.ok) {
        return response;
    }

    const resData = await response.json();
    const token = resData.token;

    localStorage.setItem('token', token);

    return redirect('/');
}
```
Then we just add the token to the requests that need to be authenticated:
```js
fetch('', {
    headers: {
        'Authorization': 'Bearer ' + localStorage.get('token')
    }
});
```

<br><br>

### Route protection
- We should not allow access to routes that require a user to be authenticated. To do so, we can add a loader to the routes that need a token to be accessed in order to redirect the user if a token is not present:
```js
{
    export function checkAuthLoader() {
        const token = localStorage.get('token');
        
        if (!token) {
            redirect('/auth');
        }
    }

    //router
    {
        path: '',
        element: '',
        loader: checkAuthLoader
    }
}
```

<br><br>

### Automatic logout
- We might want to logout the user automatically when the token expires. For that, we should ...
    - save the expiration date on localStorage when we get the token from the server.
    - use a timer to logout automatically when the token expires
```js
export function getTokenDuration() {
    const expirationDate = new Date(localStorage.get('expiration'));
    const now = new Date();
    const duration = expirationDate.getTime() - now.getTime();
    return duration;
}
export function getAuthToken() {
    const token = localStorage.get('token');

    if (!token) {
        return null;
    }

    if (getTokenDuration() < 0) {
        return'EXPIRED';
    }
    return token;
}
```
```js
const token = localStorage.get('token');
const submit = useSubmit();

useEffect(() => {
    if (!token) {
        return;
    }
    if (token === 'EXPIRED'){
        submit(null, {action:'/logout', method:'post'});
    }

    setTimeout(() => {
        submit(null, {action:'/logout', method:'post'});
    }, getTokenDuration())
}, [token, submit])
```