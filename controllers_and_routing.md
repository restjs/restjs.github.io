# Controllers
A controller is a simple javascript class, that controls your responses to the incoming requests.

If you have passed the last step ([Quick start](quick_start.md)), you have to see a folder named `controllers` in your `src` folder.

We will create and place our controllers in the `controllers` folder.
When you clone https://github.com/restjs/restjs-starter repository there's one example controller in the `controllers` folder, and as you can see in the `main.tsx` file we've imported it, also use it as a prop for the router component.

**./src/controllers/MainController.tsx**
```
export default class MainController{
    index(req, res){
        return "Welcome to Rest-JS framework!!!"
    }
}

```

**./src/main.tsx**
```
import React from 'react';
import {Application, Router, Get} from '@restjs/core';
import MainController from "./controllers/MainController";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router path="/" controller={MainController}>
            <Get path="/" handle="index"/>
        </Router>
    </Application>
)

Application.run(app);
```
In the following example we've defined method name using `handle` prop of `Get` method, this means every get request for `http://localhost:3000` will be handled by `index` method of `MainController` class.
# Router
`Router` is a react component that uses `express-js` to create a routing system.

Every `Router` could have some common methods such as `Get`, `Post`, `Put`, `Patch`, `Delete` like this : 

**./src/main.tsx**
```
import React from 'react';
import {Application, Router, Get, Post, Put, Patch, Delete} from '@restjs/core';
import MainController from "./controllers/MainController";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router path="/" controller={MainController}>
            <Get path="/" handle="index"/>
            <Post path="/" handle="create"/>
            <Put path="/" handle="put"/>
            <Patch path="/" handle="update"/>
            <Delete path="/" handle="remove"/>
        </Router>
    </Application>
)

Application.run(app);
```
Add some methods in the controller like this : 

```
export default class MainController{
    index(req, res){
        return "Welcome to Rest-JS framework!!!"
    }
    create(req, res){
        return "This is the post method";
    }
    put(req, res){
        return "This is the put method";
    }
    update(req, res){
        return "This is the patch method";
    }
    remove(req, res){
        return "This is the delete method";
    }
}
```

Then you can test these routes using [Postman](https://www.postman.com) to see the every method's result.

## Nested Routing
You can also use a Router component in another Router component to implement nested routing system.

```
import React from 'react';
import {Application, Router, Get} from '@restjs/core';
import MainController from "./controllers/MainController";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router path="/api" controller={MainController}>
            <Router path="/v1" controller={MainController}>
                <Get path="/first-method" handle="firstMethod"/>
                <Get path="/second-method" handle="secondMethod"/>
            </Router>
        </Router>
    </Application>
)

Application.run(app);
```

**./src/controllers/MainController.tsx**
```
export default class MainController{
    firstMethod(req,res){
        return "This is the first get method of api router";
    }
    secondMethod(req,res){
        return "This is the second get method of api router";
    }
}
```
> Notice : Every **Router** component shares its controller instance just with methods like `Get`, `Post`, `Patch`, `Put`, `Delete`.
>
>This means if you want to use nested routing, then you have to define a separate controller for every router. 
