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


# Configuration
There is some simple configuration steps to connect your REST-JS app to the database.

In the following example we create a database connection and then pass it as a prop of `Application` component.

## Create a connection

Open the `./src/main.tsx` file and config your app like this : 

```
import React from 'react';
import {Application, Router, Get} from '@restjs/core';
import MainController from "./controllers/MainController";
import {createConnection} from "typeorm";
import path from "path";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "",
    database: "the-database-name",
    synchronize: true,
    logging: false,
    entities : [
        /// We're loading all existing entities in the entities folder.
        path.join(__dirname, "entities/**/*.*")
    ]
}).then((connection)=>{

    const app : React.ReactElement = (
        <Application
            onListen={()=>{
                console.log('Rest-JS app is running on : http://localhost:3000');
            }}
            database={connection}
        >
            <Router controller={MainController}>
                <Get handle="index"/>
            </Router>
        </Application>
    )

    Application.run(app);

}).catch((err)=>{
    console.log(err);
});

```

>In this example we use MySQL, but you can install every database that you want and then change the `type` property of `createConnection`.

## Create an entity
Entity is your model decorated by an **@Entity** decorator. A database table will be created for such models. You work with entities everywhere with TypeORM. You can load/insert/update/remove and perform other operations with them.

We place the entities into a folder with path : `./src/entities`.

So create a file :

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

Now, a database table will be created for the Post entity and we'll be able to work with it anywhere in our app.

