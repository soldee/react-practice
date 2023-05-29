## Sending HTTP requests

- Fetch API (using Promises)
```js
function fetchData() {
    fetch('https://…')
    .then((response) => {
        return response.json(); 
    })
    .then((data) => {
        setData(data.results); 
    })
}
```

- Fetch API (using async/await)
```js
async function fetchData() {
    const response = await fetch('https://…');
    const data = await response.json();
    setData(data);
}
```

- Fetch API POST
```js
async function postData(dataObject) {
	const response = fetch('https://…', {
		method: 'POST',
	body: JSON.stringify(dataObject)
	headers: {
		'Content-Type': 'application/json'
}
});
const data = await response.json();
console.log(data);
}
```
