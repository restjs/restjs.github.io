# Pipes
<img src="/images/RestJS-framework-pipes.jpg"/>

Pipes are simple javascript classes that can be used for tow common purposes :

**1 - Validation**

**2 - Transform Data**

# Example
In the following example we show you how to use a pipe in the `Router`.

First create a file named `ExamplePipe.tsx` in this path : `./src/pipes/ExamplePipe.tsx`, then copy this code in it : 

```
import {HttpException} from "@restjs/core";

export default class ExamplePipe{
    use(req,res){
        if(!req.query.count){
            throw new HttpException("Please include 'count' property in your query.", 400);
        }
        req.query.count = parseInt(req.query.count);
    }
}
```
As you see we can have a method named `use` in the `ExamplePipe` class to handle incoming request.

Then you should include it in your router at `./src/main.tsx` like this : 

```
import React from 'react';
import {Application, Router, Get} from '@restjs/core';
import MainController from "./controllers/MainController";
import ExamplePipe from "./pipes/ExamplePipe";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router path="/" controller={MainController} pipes={[ExamplePipe]}>
            <Get path="/search" handle="search"/>
        </Router>
    </Application>
)

Application.run(app);
```
The `Router` component accepts a property of an array that includes the pipes that you want to be used for children methods.

Finally, add a method named `search` in your controller to handle the result.

**./src/controllers/MainController.tsx**
```
export default class MainController{
    search(req,res){
        return req.query;
    }
}
```

This pipes actually checks the query object of the request to have a property named 'count', if it doesn't exist then the program will throw an HttpException that will be catch by [Filters](filters.md).

We'll discuss filters in the following sessions, for now it's time to test the program to see how it works.

Open this link to see the result : 
http://localhost:3000/search

You should see this message on your browser:
```
{
  "message": "Please include 'count' property in your query.",
  "statusCode": 400
}
```
This is a common error message that is created by `HttpException`, because you've not included the 'count' property in your request's query object.

As you know query object of a request will be parsed from its url, so open the following link : 

http://localhost:3000/search?count=10

You have to see the result like this : 
```
{
  "count": 10
}
```
Absolutely the result is different, you see the 'count' property is parsed to an integer, because we've used `parseInt` in the pipe.

So, in this pipe example we checked that the query object has to include a property named 'count' (Validation), and we also transformed it to an integer (Transformation).

# Sharing
Every `Router` shares its pipes to all children such as `Router`, `Get`, `Post` and etc.
In the following code the main router with path "/" will share its pipes to the `Router` with path "/api/user", no matter how many times the routers are nested, it always shares the pipe instances with its children.
```
<Router path="/" controller={MainController} pipes={[ExamplePipe]}>
    <Get path="/search" handle="search"/>
    <Router path="/api" controller={UserController}>
        <Get path="/users" handle="users"/>
    </Router>
</Router>
```
## Ignore parent pipes
Sometimes you don't need to shared pipe instances in a child router component.
Then you can easily avoid the sharing between the nested routers by adding a prop named `ignoreParentPipes` into the router component.
```
<Router path="/" controller={MainController} pipes={[ExamplePipe]}>
    <Get path="/search" handle="search"/>
    <Router path="/api" controller={UserController} ignoreParentPipes>
        <Get path="/users" handle="users"/>
    </Router>
</Router>
```
As you can see the parent router won't share its pipes to "/api" router.

# Validation Pipe
Rest-JS offers a built-in pipe that uses another external package named `class-validator`, by using this ability we define the validation conditions in a separate class that we call it **dto(data transfer object)**.

For example create a new file with the following path : `./src/dto/CreateUserDto.tsx`.

In this example we are going to validate user's entered fields to prevent creating a user with invalid information.

Paste these codes in the created file :
```
import {Length, Matches} from "class-validator";

export default class CreateUserDto {
    @Length(3,20)
    name : string;

    @Matches(/^[a-zA-Z\-]{6,15}$/, {message : "'username' property can use only a-Z characters and - (dash) with length of 6-15 "})
    username : string;
}
```
Modify `./src/main.tsx` file like this :

```
import React from 'react';
import {Application, Router, Post} from '@restjs/core';
import UserController from "./controllers/UserController";
import CreateUserDto from "./dto/CreateUserDto";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        <Router path="/user" controller={UserController}>
            <Post
                path="register"
                handle="createUser"
                validate={{
                    body : CreateUserDto,
                    whitelist : true,
                    forbidNonWhitelisted : true
                }}
            />
        </Router>
    </Application>
)

Application.run(app);
```
>Notice : `whitelist` and `forbidNonWhitelisted` properties are the class-validator's options, that tells to class-validator to don't pass other properties of body that are not listed in the dto class.
>
>Refer to [class-validator](https://github.com/typestack/class-validator) docs for more information .

 

Then create a controller to handle the validated data like this :

```
import CreateUserDto from "../dto/CreateUserDto";

export default class UserController{
    createUser(req,res){
        const user : CreateUserDto = req.body;
        console.log("new user : ",user);
        /// do something with the validated data (user)

        return "The user's data is valid and can be saved.";
    }
}
```
Now save the changes and run `npm run dev` to run the development server.
Then open Postman and test the written api on : http://localhost:3000/user/register (Post method).

If you enter `name` and `username` property of body with correct format then you should see this message : 
```
The user's data is valid and can be saved."
```
That means the request has passed of the validation pipe and now you can use the validated data in the defined controller's method.

Otherwise, if you enter `name` and `username` in an incorrect format or do not include one of them, you should see an error message like this : 
```
{
    "message": [
        "name must be longer than or equal to 3 characters",
        "'username' property can use only a-Z characters and - (dash) with length of 6-15 "
    ],
    "statusCode": 400
}
``` 


