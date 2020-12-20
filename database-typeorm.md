TypeORM is an ORM library that built top on TypeScript.
TypeORM supports "Active Record" and "Data Mappern" patterns which means you can write high quality, scalable, maintainable applications.

TypeORM supports some popular databases : MySQL, MariaDB, MongoDB, PostgreSQL, CockroachDB, SQLite, Microsoft SQL Server, sql.js, Oracle, etc.

REST-JS is supporting TypeORM since v 0.0.8, you can easily config your database and access it into your app's services by using DI(dependency injection) design pattern .

You can read more about TypeORM on : https://typeorm.io

# Installation
1 - Install the npm package : 

`npm install typeorm --save`

2 - Install the database driver : 

for **MySQL** or **MariaDB** :

`npm install mysql --save` 

(you can install mysql2 instead as well)

for **PostgreSQL** or **CockroachDB**:

`npm install pg --save`

for **SQLite**:

`npm install sqlite3 --save`

for **Microsoft SQL Server**:

`npm install mssql --save`

for **sql.js**:

`npm install sql.js --save`

for **Oracle**:

`npm install oracledb --save`

To make the Oracle driver work, you need to follow the installation instructions from their site.


for **MongoDB (experimental)**

`npm install mongodb --save`


# Example
In the following example we show you how to create a simple blog rest apis to create, update and show posts.

Note that we won't implement an authentication system on this example, we'll discuss the authentication on the next session.

You can clone this example's source code on : 

https://github.com/restjs/restjs-typeorm-example

## Create a connection

Open the `./src/main.tsx` file and config your app like this : 

```
import React from 'react';
import {Application} from '@restjs/core';
import {createConnection, Connection} from "typeorm";
import PostEntity from "./entities/PostEntity";
import PostRouter from "./PostRouter";

/**
 * ::: Notice :::
 * This example only show how to work with databases and validation system.
 * In the real world you should specify an authentication middle-ware to handle the user's posts.
 * We will discuss the authentication on the next session.
 */

createConnection({
    /// In this example we used MySQL database, you can use every database that is supported by TypeORM.
    type : "mysql",
    database : "rest", /// Your database's name
    username : "root", /// Your database's username
    password : "",     /// Your database's password
    entities : [PostEntity], /// Load the entities here, that you'll use it into your project.
    synchronize : true, /// This option is used to automatically generate the tables into the database.
}).then((connection : Connection)=>{

    const app : React.ReactElement = (
        <Application
            onListen={()=>{
                console.log('Rest-JS app is running on : http://localhost:3000');
            }}
            database={connection}
        >
            {PostRouter}
        </Application>
    )

    Application.run(app);


}).catch((error=>{
    /// Handle the error on connecting to database.
    throw new Error(error);
}))

```

>In this example we use MySQL, but you can install every database that you want and then change the `type` property of `createConnection`.

## Create a router
It's a good practice to define your routers in some separate files to clean up `main.tsx` file and make your code more readable.

**./src/PostRouter.tsx**
```
import React from "react";
import {Get, Patch, Put, Router} from "@restjs/core";
import PostController from "./controllers/PostController";
import CreatePostDto from "./dto/CreatePostDto";
import UpdatePostDto from "./dto/UpdatePostDto";
import PostService from "./services/PostService";

export default (
    <Router
        path="post"
        controller={PostController}
        services={[PostService]}
    >

        <Get handle="allPosts"/>

        <Get path="published" handle="publishedPosts"/>

        <Put
            handle="create"
            validate={{
                /// Setting CreatePostDto for body property of every request object.
                body : CreatePostDto,

                /**
                 * `whitelist` and `forbidNonWhitelisted` are the `class-validator` options .
                 * More information at :
                 * https://github.com/typestack/class-validator
                 */
                whitelist : true,
                forbidNonWhitelisted : true,

                /**
                 * the transform property is used to transform strings to its dto type.
                 * Such as id property, id should be an integer.
                 * `class-transformer` package will transform plain simple object data to a class.
                 * Refer to class transformer documentation for more information :
                 * https://github.com/typestack/class-transformer
                 */
                transform : { enableImplicitConversion : true }
            }}
        />

        <Patch
            handle="update"
            validate={{
                body : UpdatePostDto,
                whitelist : true,
                forbidNonWhitelisted : true,
                transform : { enableImplicitConversion : true }
            }}
        />
    </Router>
)

```
We used tow DTOs (CreatePostDto, UpdatePostDto) to validate and transform the incoming request body, it's a good practice to define some separate DTOs for every operation. If you take a look at [example's source code](https://github.com/restjs/restjs-typeorm-example) you can see them in the `/src/dto` folder.

This session is about the database not the validation & transformation, You can read more about [Transformation & Validation here](pipes.md).

## Create an entity
Entity is your model decorated by an **@Entity** decorator. A database table will be created for such models. You work with entities everywhere with TypeORM. You can load/insert/update/remove and perform other operations with them.

We place the entities into a folder with path : `./src/entities`.

**./src/entities/PostEntity.ts**
```
import {BaseEntity, Column, Entity, PrimaryGeneratedColumn} from "typeorm";

@Entity()
export default class PostEntity extends BaseEntity{
    @PrimaryGeneratedColumn()
    id : number;

    @Column()
    title : string;

    @Column()
    content : string;

    @Column({default : false})
    isPublished: boolean;
}

```
> Note that **Entity** is a decorator, but **BaseEntity** is a class that **PostEntity** inherits from it.

Now, a database table will be created for the Post entity, and we'll be able to work with it anywhere in our app.

## Create a service

As you know [Services](services.md) are used to provide the data for controllers, so we'll create a new service to access the database repository.

**/src/services/PostService.ts**
```
import {InjectRepository} from "@restjs/core";
import {Repository, UpdateResult} from "typeorm";
import PostEntity from "../entities/PostEntity";
import UpdatePostDto from "../dto/UpdatePostDto";
import CreatePostDto from "../dto/CreatePostDto";

export default class PostService {
    @InjectRepository(PostEntity)
    postRepository : Repository<PostEntity>;

    async create(post : CreatePostDto) : Promise<PostEntity>{
        const createdPost : PostEntity = this.postRepository.create(post);
        await createdPost.save(); /// Saving the created post into the database.
        return createdPost;
    }

    update(postId : number, post : UpdatePostDto) : Promise<UpdateResult>{
        return this.postRepository.update(postId, post);
    }

    getAll() : Promise<PostEntity[]>{
        return this.postRepository.find();
    }

    publishedPosts() : Promise<PostEntity[]>{
        return this.postRepository.find({where : {isPublished : true}});
    }
}
```
> @InjectRepository decorator is used to inject TypeORM's repositories into the service.

## Create a controller
Finally, your app needs a controller to use the post's service to respond .

**/src/controllers/PostController.ts**

```
import {Inject} from "@restjs/core";
import PostService from "../services/PostService";
import PostEntity from "../entities/PostEntity";
import {UpdateResult} from "typeorm";

export default class PostController{
    @Inject()
    postService : PostService;

    /**
     * REST-JS supports promises.
     * So every method can return a promise and REST-JS will resolve it and then responds.
     */
    create(req) : Promise<PostEntity>{
        /**
         * req.body is validated by class-validator and also transformed by class-transformer.
         * So we can use req.body without worrying about that because it's validated by the DTO (Data transfer object).
         */
        return this.postService.create(req.body);
    }

    /// When we update some entity, TypeORM will return an instance of UpdateResult.
    update(req) : Promise<UpdateResult>{

        /// First we extract the id of the dto and the other Post's data properties.
        const {id, ...postData} = req.body;

        return this.postService.update(id, postData);
    }

    /// You can also use async/await syntax.
    async allPosts() : Promise<PostEntity[]>{
        return await this.postService.getAll();
    }

    publishedPosts() : Promise<PostEntity[]>{
        return this.postService.publishedPosts();
    }

}

```

The app is ready! You can test it using `PostMan`.
Ths post's routes to test :

Get all posts list : http://localhost:3000/post (GET)

Create a new post : http://localhost:3000/post (POST)

Modify a post : http://localhost:3000/post (PATCH)

Get published posts : http://localhost:3000/post/published (GET)

