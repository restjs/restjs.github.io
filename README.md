# NodeJS
Skip this section if you know about NodeJS.

In the past years NodeJS have been used by many of big companies and developers.
NodeJS applications are quick and scalable .

NodeJS uses Non-blocking execution method to make sure the process never will be stop when many of calculations are running on your server.

So you can use NodeJS to build server-side applications, just by using JavaScript.
But making a server-side application only with pure JavaScript is a huge challenge and makes a lot of headaches.

Fortunately NodeJS has a great developer community and lots of packages that are written by other developers worldwide, and you can use these packages with a command line application named `Node package manager(NPM)`,Please refer to https://npmjs.org to get more information about that.

If you've worked with ReactJS, you might know about JSX syntax, JSX syntax is like HTML but in the fact it's JavaScript code that will be compiled by a NodeJS package called babel.

We use React (for JSX syntax) and Express packages to configure our NodeJS routing system in a simple way.

# What is Rest-JS?

Rest-JS is a simple and lightweight framework heavily inspired by Nest-JS framework to build Node-JS applications in some simple steps. 

Rest-JS implements some common design patterns like : MVC, Dependency Injection (DI), Pipelines, Interceptors and etc. It also uses OOP (Object-oriented programming) and FP(functional programming) to make sure you'll build performant and scalable server-side applications. 

You can deploy your NodeJS server using JSX syntax like this : 
```
import React from 'react';
import {Application, Router, Get} from '@restjs/core';
import MainController from './controllers/MainController';

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


# Prerequisites

Before start working with Rest-JS, please make sure you know some basics about the mentioned technologies :

1 - NodeJS (https://nodejs.org)

2 - ReactJS (https://reactjs.org)

3 - ExpressJS (https://expressjs.com/)

4 - Typescript (https://www.typescriptlang.org)
