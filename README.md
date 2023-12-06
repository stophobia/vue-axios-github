[![Travis](https://img.shields.io/travis/superman66/vue-axios-github.svg)](https://travis-ci.org/superman66/vue-axios-github)
# A project learns how to implement login interception on the front end

> A project learns to use vue family bucket + axios to implement login, interception, and logout functions, and to use axios's http interceptor to intercept requests and responses.

Click here to view 👉 [Demo](https://github.com/superman66/vue-axios-github)

Can't open the page? Maybe you need to jump over the wall. [Click to view circumvention recommendations](https://github.com/superman66/mac/issues/4)
## Preface
This project uses the personal token provided by Github as a login token, and accesses your Repository List through the token. Learn how to implement what is needed in a front-end project through this project
Login and interception, logout, token invalidation interception and the use of the corresponding axios interceptor.

**Prepare**

You need to generate your own Github Personal Token first ([Generate Token](https://github.com/settings/tokens/new)).
After the Token is generated, visit [Demo](http://chenhuichao.com/vue-axios-github/) to view your Repository List.

## Project structure

```
.
├── README.md
├── dist  // Package the built folder
│   ├── build.js
│   └── build.js.map
├── index.html
├── package.json
├── src
│   ├── App.vue
│   ├── assets
│   │   ├── css.css
│   │   ├── icon.css
│   │   └── logo.png
│   ├── constant
│   │   └── api.js  // Configure api interface file
│   ├── http.js // Encapsulate fetch, post requests and http interceptor configuration files
│   ├── index.vue
│   ├── login.vue
│   ├── main.js
│   ├── repository.vue
│   ├── router.js // routing configuration file
│   └── store
│       ├── store.js  
│       └── types.js  // vuex types
└── webpack.config.js
```

### technology stack
* Vue 2.0
* vue-router
* vuex
* axios
* vue-material

### Login interception logic

#### Step one: Route interception
First, when defining a route, you need to add a custom field `requireAuth` to determine whether access to the route requires login. If the user is already logged in, the route will be entered successfully.
Otherwise, go to the login page.
```javascript
const routes = [
    {
        path: '/',
        name: '/',
        component: Index
    },
    {
        path: '/repository',
        name: 'repository',
        meta: {
            requireAuth: true,  // Add this field to indicate that you need to log in to enter this route
        },
        component: Repository
    },
    {
        path: '/login',
        name: 'login',
        component: Login
    }
];
```
After defining the route, we mainly use the hook function `beforeEach()` provided by `vue-router` to judge the route.

```javascript
router.beforeEach((to, from, next) => {
    if (to.meta.requireAuth) {  // Determine whether the route requires login permissions
        if (store.state.token) {  // Get whether the current token exists through vuex state
            next();
        }
        else {
            next({
                path: '/login',
                query: {redirect: to.fullPath}  // Use the jump route path as a parameter and jump to this route after successful login.
            })
        }
    }
    else {
        next();
    }
})
```
Each hook method receives three parameters:

* to: Route: The target routing object that is about to enter
* from: Route: The route that the current navigation is about to leave
* next: Function: This method must be called to resolve this hook. The execution effect depends on the calling parameters of the next method.
  * next(): Perform the next hook in the pipeline. If all hooks are executed, the navigation status is confirmed.
  * next(false): Interrupt current navigation. If the browser's URL changes (perhaps manually by the user or by the browser's back button), the URL address will be reset to the address corresponding to the from route.
  * next('/') or next({ path: '/' }): Jump to a different address. The current navigation is interrupted and a new navigation is started.

**Make sure to call the next method, otherwise the hook will not be resolved. **
> For the complete method, see `/src/router.js`

Among them, `to.meta` contains our customized data, including the `requireAuth` field we just defined. Use this field to determine whether the route requires login permissions. If necessary, and if there is no token in the current application, jump to the login page and log in. After successful login, jump to the target route.

Does login interception end here? not at all. This method is just a simple front-end routing control and cannot really prevent users from accessing routes that require login permissions. There is another situation: the current token has expired, but the token is still saved locally. At this time, when you access a route that requires login permissions, you should actually ask the user to log in again.
At this time, it needs to be judged by combining the http status code returned by the http interceptor + the backend interface.

#### Step 2: Interceptor
If you want to handle all http requests and responses uniformly, you have to use the axios interceptor. By configuring `http response inteceptor`, when the backend interface returns `401 Unauthorized (unauthorized)`, the user is allowed to log in again.

```javascript
// http request interceptor
axios.interceptors.request.use(
    config => {
        if (store.state.token) {  // Determine whether there is a token. If it exists, add the token to each http header.
            config.headers.Authorization = `token ${store.state.token}`;
        }
        return config;
    },
    err => {
        return Promise.reject(err);
    });

// http response Interceptor
axios.interceptors.response.use(
    response => {
        return response;
    },
    error => {
        if (error.response) {
            switch (error.response.status) {
                case 401:
                    // Return 401, clear token information and jump to the login page
                    store.commit(types.LOGOUT);
                    router.replace({
                        path: 'login',
                        query: {redirect: router.currentRoute.fullPath}
                    })
            }
        }
        return Promise.reject(error.response.data)   // Returns the error information returned by the interface
    });
```
>See `/src/http.js` for the complete method.

Through the above two steps, login interception can be implemented on the front end. The `logout` function is very simple. You only need to clear the current token and jump to the homepage.


## Run and build
``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```
