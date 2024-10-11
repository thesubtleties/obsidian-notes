## Installation

```bash
npm install axios
# or
yarn add axios
```

## Basic Usage

```javascript
import axios from 'axios';

// GET request
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response.data);
  })
  .catch(function (error) {
    console.log(error);
  });

// POST request
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

## Creating an Instance

```javascript
const instance = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

## Request Config

```javascript
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```

## Response Schema

```javascript
{
  data: {},
  status: 200,
  statusText: 'OK',
  headers: {},
  config: {},
  request: {}
}
```

## Interceptors

```javascript
// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Any status code that lie within the range of 2xx cause this function to trigger
    return response;
  }, function (error) {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    return Promise.reject(error);
  });
```

## Cancellation

```javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function (thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // handle error
  }
});

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.');
```

## Async/Await Usage

```javascript
async function getUser() {
  try {
    const response = await axios.get('/user?ID=12345');
    console.log(response);
  } catch (error) {
    console.error(error);
  }
}
```

Key Features of Axios:
1. Makes XMLHttpRequests from the browser
2. Makes http requests from node.js
3. Supports the Promise API
4. Intercept request and response
5. Transform request and response data
6. Cancel requests
7. Automatic transforms for JSON data
8. Client side support for protecting against XSRF

Axios is often preferred over the native Fetch API because it has a more powerful and flexible feature set, automatic request and response transformations, and better error handling out of the box.