# Unit 10 Databases

## Summary

In this unit you have will have a backend Node/Express server interact with a database to provide data to a pre-built frontend (react application).

**Note:** For this unit, you will be working in two different branches to implement SQL and NoSQL versions of the challenge. When you fork this repo, please make sure to uncheck the "Copy the `main` branch only" setting so that your forked copy will also contain the `mongoose` branch.

## Learning goals

- Understand the role of databases and how we can set up our backend to retrieve and store data from and to databases.
- Learn the difference between SQL and NoSQL(document store) databases and when you might use one or the other.
- Set up a remote cloud PostgreSQL database on ElephantSQL.
- Use node module `node-postgres` (or `pg`) to interact with a PostgresQL database.
- Set up a remote cloud Mongo database on MongoDB Atlas.
- Use node module `mongoose` to interact with a Mongo database.

## What is node-postgres?

You are already familiar with PostgreSQL from having worked on the skill builder for this unit. Node-postgres is a node module published on npm that provides functions to access a PostgreSQL database whether it is hosted locally (on your machine), or remotely (in the cloud). It lets you send SQL queries from a Node server to the database and provide you the response received from the database. Documentations on how to use it may be found [here](https://node-postgres.com/).

## What is MongoDB/mongoose?

MongoDB is the most popular non-relational database in use today. It is a document-oriented database that can store data in the form of documents, which are similar to objects in javascript. Instead of _tables_ in SQL, we have _collections_. Instead of _columns_ in SQL, we have _keys_. Instead of _rows_ in SQL, we have _documents_.

NoSQL databases like MongoDB are inherently more flexible than SQL databases as they do not need to follow a model. While a SQL table can only contain rows that conform to the specific schema of columns, collections in a NoSQL database can contain documents with different kinds of data that don't neatly fit into a schema. They may even contain arrays and objects that are nested.

Mongoose is a node module published on npm that not only provides a way to access a MongoDB database from a node server but also add a data schema to MongoDB collections. While not being beholden to a schema may be a feature of a NoSQL database, it can also be a cause of confusion and error if data is not organized in a predictable structure. Mongoose aims to solve that by letting developers create schemas and use models created off of that schema to interact with MongoDB.

Documentations for Mongoose can be found [here](https://mongoosejs.com/docs/guide.html).

## What will be the challenge?

We will be creating a database in the cloud and fill it with Star Wars data. Then, we will be working off of a slightly modified version of the app from the previous Express Unit, configuring the backend to retrieve a list of characters from the database and send it to the frontend to display. We will also be retrieving additional information about a character's species, homeworld, and films they've been in from the database. Finally, we will be adding the ability to add a new character into the database. We will do this with both PostgreSQL and MongoDB.

You do not need to modify any frontend code, only the backend, but feel free to look around if you are curious.

<hr />

## Challenges

### Part 1: PostgreSQL/node-postgres

#### Set up / Install

1. [ ] You should already have an ElephantSQL database in the cloud with the Star Wars database loaded from the [Unit 10 Skill Builder](https://github.com/CodesmithLLC/unit-10SB-databases). We will be using the same database so be sure to have the connection URL handy.
1. [ ] Install your npm dependencies by running `npm install` in your terminal.
1. [ ] Run `npm run dev` to start your server and bundle the frontend. It will launch the frontend application in a browser window which will currently have no characters.
   - NOTE: If you are having trouble loading the page, run `node -v` to get the version of node installed on your computer and ensure you have the [latest stable (LTS) version](https://nodejs.org/en/) and not an unstable version.
     If you have an unstable version, run `npm install -g n` to install n globally - `n` allows us to interact with different Node.js versions. Next, run `sudo n stable` to downgrade to the latest stable version or `sudo n versionnumber` to download a specific version of node.

#### Set up your database connection

1. [ ] Take a look into `server/models/starWarsModels.js`. Add your connection URL to the ElephantSQL database into `PG_URI`.
1. [ ] We will be using this connection string to connect to the database via a pool. You can see the syntax [here](https://node-postgres.com/features/connecting#Connection%20URI).
   - We are using a pool to manage our connections to the database which has a few advantages. There is overhead in establishing new connections to the database that we don't want to wait on for every single query we send. Having a pool of connections at the ready makes our database queries more performant. Read more about it [here](https://node-postgres.com/features/pooling).
1. [ ] Now we are ready to send queries using `pool.query()` to the database. At the bottom of this file, we are exporting an object with a property called query, which is a function that will return the invocation of `pool.query()`. In between this, we can add `console.log` for all the queries being made for tracking purposes.
   - NOTE: While we could export the pool from here and just use `pool.query()` directly from the controller, exporting this way lets us control all the database queries from one file location, and any logic or logging that needs to change lives in this one file.
1. [ ] Here is the ER diagram of the database again to reference for the upcoming sections:
       ![pg_schema](/docs/assets/images/schema.png)

#### Get and serve characters

1. [ ] On load, the frontend will be making a GET request to `/api/` to get an array of characters. Check out the route handlers in the `server/routes/api.js` file. Notice it is using `starWarsController.getCharacters` as a middleware, and afterwards sending a JSON response with an empty array.
1. [ ] Look in the `server/controllers/starWarsController.js` file. We have imported our entry to the database from `server/models/starWarsModels.js` as `db`. Fill out the `starWarsController.getCharacters` using `db.query()` to get a list of all the characters inside our **people** table.
   - A good way to structure your code to be a bit cleaner here is to first declare a variable that will contain the SQL query string and pass that variable into `db.query()` as an argument instead of writing the whole SQL query directly inside the `db.query()`. Let's start by writing and sending a query to grab all the columns from the **people** table.
   - The syntax for using `pool.query()`, and therefore our `db.query()` which returns its invocation, supports both passing in a callback to be run after the query is sent, or it will return a promise if no callback is passed in so you can `.then()` on it.
   - You may want to `console.log` the result you get back from the database to see what it looks like. Take the relevant result of the query and store it in `res.locals` so that it may be passed along to the next middleware. Then call `next()` to move on to the next middleware in the route handler.
   - Be mindful of asynchronicity here. `pool.query()` is asynchronous so make sure you are moving onto the next middleware at the right time (after the query results are back).
   - Also practice good error handling. If the database query results in an error, we want to use the express global error handler by calling `next` with an error object passed in.
1. [ ] Back in the `server/routes/api.js` file, finish out the `/api/` GET route by sending the query results stored in `res.locals` as JSON response to the frontend.
   - The frontend is expecting to receive an array of objects so make sure your response matches that.
   - Refreshing the frontend app should now display character cards based on the data that was passed.
1. [ ] Take a closer look at the character cards though. There will be some information missing like homeworld and species. To be more specific, the frontend is expecting each character object in the array we send back to contain the following properties: name, gender, species, species_id, birth_year, eye_color, hair_color, homeworld, homeworld_id, and films (which should be an array of objects with keys: title and id). We will need to modify our SQL query to gather all the relevant data.
   - In order to collect all the data required for this response, we will need to adjust our SQL query to pull data from multiple tables besides **people**. It may be helpful to consult the ER Diagram of all the tables and columns in our database given above.
   - HINT: _species_ will come from the _name_ of the species (as _species_) corresponding to _species_id_ in the **species** table and _homeworld_ will come from the _name_ of the planet (as _homeworld_) corresponding to the _homeworld_id_ in the **planets** table. Can we write a query to make sure we select everything from the **people** table and fill in additionally from the **species** table and the **planets** table? It could be helpful to try out the queries in ElephantSQL's browser, pgAdmin, or the `psql` command line as you did for the skill builder.
   - Check the frontend again by refreshing to see if the missing details have been populated.
1. [ ] (BONUS) Getting a list of films a character has been in will require using the join table of **people_in_films**. You will have to run another query for each character to generate an array of films that each character has been in. Remember, you'll need an object with the _title_ and the _\_id_ (as _id_ without the underscore) for each film in the array. Since you have to make a new query for each character, this will take quite some time to generate the response. Feel free to skip this one for now and come back to this later if you can.

#### Get and serve species details

1. [ ] You'll notice in the frontend character card, next to the species is a blue question mark which when click opens up a modal. However, the detailed data is not yet populated. The frontend makes a GET request to `/api/species` for this information. The GET request will include the id of the species in a query string like `/api/species?id=1`.
1. [ ] The route handler is already there in the `server/routes/api.js` file but we will have to complete the `starWarsController.getSpecies` middleware in `server/controllers/starWarsController.js`.
   - Grab the id from the **req**uest's query string and save it to a variable.
   - Write a SQL query to get the classification, average_height, average_lifespan, language, name, and homeworld of the species using this id. Check the `node-postgres` documentations as to how to pass in a [parameterized query](https://node-postgres.com/features/queries#Parameterized%20query) so that you can pass in the `_id` safely into the SQL query.
   - HINT: Getting the name of the homeworld will require a join to get the _name_ of the _planet_ as _homeworld_.
   - Once you've obtained the data, save it for the next middleware, and send a JSON response to the frontend an object with all the properties mentioned above.
   - Check the frontend to see if the modal now populates with the appropriate data for each species.
   - NOTE: We are now sending a query based on some condition. Be aware that if the database does not find any data matching the provided condition, it will **not** result in an error. Instead, the query is considered successful with an empty result. If you are wanting to catch cases where you don't find a certain data from the database, you must deliberately include that logic in the callback or `.then()`.

#### Get and serve homeworld details

1. [ ] Much like the the species above, the homeworld also has a question mark for a modal with details. A GET request will be made to `/api/homeworld` with a query string providing the id.
1. [ ] Complete the `starWarscontroller.getHomeworld` middleware.
   - Write a SQL query to get the planet's rotation_period, orbital_period, diameter, climate, gravity, terrain, surface_water, and population.
   - Send a JSON response to the frontend with an object with the above properties, and check to see if the modal populates with the corresponding data.

#### (BONUS) Get and serve film details

1. [ ] If you have gotten the bonus to get an array of films to the frontend, you will also see the list of films with question marks on the frontend. A GET request will be made to `/api/film` with a query string providing the id, and it will be expecting a response of an object with following properties: title, episode_id, director, producer, and release_date.

#### Add a new character

1. [ ] On the top-right corner of the frontend app is a button labeled "Create Character". Clicking it will take you to a character creation page. Filling out the form and clicking on the "Save" button will send a POST request to the backend to `/api/character`. This POST request will include a body with the following keys: name, gender, species, species_id, birth_year, eye_color, skin_color, hair_color, mass, height, homeworld, homeworld_id, films. Feel free to `console.log` the request body to see what kinds of value they contain.
1. [ ] In `server/controllers/starWarsController.js`, complete the `starWarsController.addCharacter` middleware to take all these values from the request body and use them to insert a new person into the **people** table.
   - You will not need all the data that is being sent from the frontend so only pick the relevant ones when formulating your insert parameterized query.
   - The server does not need to send any data back besides a 200 status but feel free to send back the inserted row if you would like. (Your SQL query should specify that you want the inserted data returned)
   - If successful, you should see your new character show up at the end of the character list on the frontend, or you can also look it up from your database directly via the ElephantSQL browser, pgAdmin, or the psql command line.
   - (BONUS) Also insert rows to the **people_in_films** table based on the film selection being sent from the frontend. HINT: You will need the _\_id_ of the person that just got inserted to do this.

#### Extensions (Only work on these if you finished Part 2)

1. [ ] Try to make additional routes and controllers to update and delete records from the database. While we do not have a frontend to make those requests, you can still use Postman to hit those endpoints and trigger the controller.
1. [ ] Checkout to a new branch and refactor your code to use the node module `sequelize`, an ORM (Object Relational Mapping) library which provides another way for a NodeJS server to interact with SQL databases. See it [here](https://sequelize.org/master/).

### Part 2: MongoDB/mongoose

Make sure you have committed your work from working on pg above. Then we will switch to a branch set up for this part by running the command `git checkout mongoose` in your terminal.

The instructions for this part of the challenge can be found in the `mongoose` branch.
