# Filter

<img src="/images/RestJS-framework-filters.jpg"/>

The Filter is a nice feature for REST-JS that can handle error messages that maybe occurs in your application.

If you passed the last sessions you maybe have seen HttpException class that we use to respond to the user with an error message.

For example, we throw a new error (HttpException) in the `MainController` to tell the user that you are not logged in :

```
import {HttpException} from "@restjs/core";

export default class MainController{
    userInfo(req){
        if(req.hasOwnProperty("user")){
            throw new HttpException("unauthorized",401);
        }
        return {user : req.user};
    }
}
```  
`MainController` has a method that checks `req` object , if `req` has a property as `user` then returns it as a success response with success status (200), otherwise it will throw a new `HttpException` with 401(Unauthorized) code.

# Another Example
Let's have an example to see how filters work.
You can get this example's source code on : https://github.com/restjs/restjs-filters-example

In the following example we implement a solution to translate thrown `HttpException` messages .
First we create a class on `./src/filters/TranslationFilter.tsx`.
```
import React from "react";
import ReactDOMServer from "react-dom/server";
interface ErrorMessageInterface {
    message : string | Array<string>;
    statusCode : number;
    out? : any;
}

export default class TranslationFilter{
    source = {
        "unauthorized" : "You are not logged in !",
        "not_found" : "This page is removed ."
    }

    translate(key : string){
        const findSource = this.source[key];
        if(findSource){
            return findSource;
        }
        return key;
    }

    catch(message, statusCode : number) : ErrorMessageInterface{
        if(typeof message == 'string'){
            message = <h2 style={{color:"red"}}>{this.translate(message)}</h2>;
        }else if(Array.isArray(message)){
            message = message.map(m=>this.translate(m));
        }
        message = ReactDOMServer.renderToStaticMarkup(message);
        return({message, statusCode, out : "text"});
    }
}

```
There's a method at class `TranslationFilter`, that will catch thrown `HttpException`s of the application.

You can transform error's message or status code and return a new object to transform the error.

In this example we use `react-dom/server` to render the data into a plain HTML string.

The returned object from the `catch` method also has a property named `out` that tells REST-JS to response immediately without effecting other next filters.
`out` property can be "text" or "json" that specifies the content type of the response, if you don't mention `out` property, then REST-JS will affect other filters.

We created a simple filter class, now we have to apply it into the router in `./src/main.tsx` .
```
import React from 'react';
import {Application, Router, Get, Post} from '@restjs/core';
import MainController from "./controllers/MainController";
import TranslationFilter from "./filters/TranslationFilter";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router
            path="/"
            controller={MainController}
            filters={[TranslationFilter]}
        >
            <Get path="/" handle="index"/>
            <Get path="/about-us" handle="deletedPage"/>

        </Router>
    </Application>
)

Application.run(app);
```

Then we create a controller to throw an error for `/about-us` route:

**./src/controllers/MainController.tsx**

```
import {HttpException} from "@restjs/core";

export default class MainController{
    index(req, res){
        return "Welcome to Rest-JS framework!!!"
    }

    deletedPage(){
        throw new HttpException("not_found",404);
    }
}
```

If you open http://localhost:3000/about-us, you should see a red message like : **This page is removed .** that means the filter works correctly.


