# Services
<img src="/images/RestJS-framework-services.jpg"/>

Services are simple javascript classes that are commonly used to fetching data from the database for the controllers .

REST-JS uses dependency injection (DI) to inject a service to a controller, so we can access a class (service) in our controller.

# Example
In the following example we implement a solution to create a system that is able to let users to register.

You can clone this example's git repository on : https://github.com/restjs/restjs-services-example .

First we create a router component with path `/user` in the following path :

**./src/UserRouter.tsx**
```
import React from "react";
import {Router, Get, Post} from '@restjs/core';
import UserController from "./controllers/UserController";
import UserService from "./services/UserService";
import ParseIdPipe from "./pipes/ParseIdPipe";
import CreateUserDto from "./dto/CreateUserDto";

export default (
    <Router path="/user"
            controller={UserController}
            services={[UserService]}
            pipes={[ParseIdPipe]}
    >
        <Post
            path="/register"
            handle="register"
            validate={{
                body : CreateUserDto,
                whitelist : true,
                forbidNonWhitelisted : true
            }}
        />

        <Get
            path="/all"
            handle="allUsers"
        />

        <Get
            path="/:id"
            handle="findUser"
        />

        {
            /**
             * In some nested routers, REST-JS always shares just one instance of every service, not different service instances.
             **/
        }
        <Router path="/api" controller={UserController}>
            <Get
                path="/all"
                handle="allUsers"
            />
        </Router>
    </Router>
)
```
This router has 3 main methods that are used for the registration, getting all users list, getting a specific user by its `id` .

The router should be imported in `./src/main.tsx` like this :

```
import React from 'react';
import {Application} from '@restjs/core';
import UserRouter from "./UserRouter";

const app : React.ReactElement = (
    <Application
        onListen={()=>{
            console.log('Rest-JS app is running on : http://localhost:3000');
        }}
    >
        {UserRouter}
    </Application>
)

Application.run(app);
```

Then we create our `UserService` :

**./src/services/UserService.tsx**

```
import {HttpException} from "@restjs/core";

export interface UserInterface {
    id? : number;
    name? : string;
    last_name? : string;
    email? : string;
}

export default class UserService{
    users : UserInterface[] = [];

    createUser(userData : UserInterface) : UserInterface{
        /// Check if the email exists, then throw a HttpException
        const findExisting = this.users.find( user => user.email == userData.email);
        if(findExisting){
            throw new HttpException("This email address is already registered, please try another one.", 409);
        }

        /// Creating an user with incremental id.
        userData.id = this.users.length + 1;
        this.users.push(userData);
        return userData;
    }

    /**
     * REST-JS supports async/await and promises.
     * You can use async methods and promises to handle asynchronous requests .
     * For example, when your program will fetch the data from the database, you have to use async functions or promises.
     * In this example we just set a timeout function to simulate the async code function.
     * This function returns the data after 2 seconds (2000 milliseconds) .
     */
    getAll() : Promise<UserInterface[]>{
        return new Promise((resolve, reject)=>{
            setTimeout(()=>{

                if(this.users.length == 0){
                    /// If you want to throw an exception into a promise, you have to use the `reject` function.
                    reject(
                        new HttpException("No user found.", 404)
                    );
                }
                resolve(this.users);
            },2000);
        })
    }

    findOne(id : number) : UserInterface{
        const user = this.users.find(user=>(user.id == id));
        if(!user){
            /// If no user found
            throw new HttpException(`No user found with id : ${id}`, 404);
        }

        /// If the user found
        return user;
    }
}
```

This class has one property named `user`, we store the user's data into this property, but in the real-world we use the database to provide and store the data.

`UserService` will be instantiated at run-time automatically by REST-JS and will be injected into the `UserController`.

Let's create a controller as `UserController` : 

**./src/controllers/UserController.tsx**
```
import {Inject} from "@restjs/core";
import UserService from "../services/UserService";

export default class UserController{
    /**
     * We use @Inject() decorator to inject an instance of the UserService class into our controller.
     * Please refer https://www.typescriptlang.org/docs/handbook/decorators.html to get more information about decorators.
     */
    @Inject()
    userService : UserService;

    register(req){
        /// The body object of req is validated by CreateUserDto
        return this.userService.createUser(req.body);
    }

    allUsers(){
        return this.userService.getAll();
    }

    findUser(req){
        return this.userService.findOne(req.params.id);
    }
}
```

As you can see we use a decorator named **@Inject()** to inject the `UserService` into `UserController`.


