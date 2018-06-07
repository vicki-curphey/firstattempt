# How To Start A Fullstack Development Environment

This set up will include Npm, Node.js, Express, MongoDb, React

## Steps to start

- [ ] Create folder & open Vs in folder
- [ ] Ensure the folder name has no spaces between it
- [ ] Once in vs pres ctrl then ~ to open terminal
- [ ] Type `node -v` & `npm -v` to ensure node.js is set up correctly

## Add package.js

CD into the folder you just created to install the package.json file

```
$ npm init
```

## Connect dev code to Github

Type each query in one at a time to ensure you catch an error if one pops up

```
$ git init
```

```
$ git add .
```

```
$ git commit -m "First commit"
```

```
$ git remote add origin (remote repository URL -> Ex: https://github.com/Muppet-training/ninjaRecipe.git)
```

```
$ git remote -v
```

```
$ git push origin master
```

Create `.gitIgnore` file to prevent certain folders & files from being uploaded into the Github repo

```
$ echo "node_modules" >> .gitignore
```

## Get MongoDb running in the background

Install it following [this tutorial](https://www.youtube.com/watch?v=A5fiWcVcADw&index=2&list=PL4cUxeGkcC9jBcybHMTIia56aV21o2cZ8)
Go to the folder where mongod.exe is located
Then type a similar query to this to get the database running

```
C:\mongodb\bin> $ mongod.exe
```

## Install Express

Adding `--save` to the query ensures the package is added to the "dependencies": in `package.json`

```
$ npm install express --save
```

## Add index.js + Configuration Code

In the `package.json` file it says we need to launch the app from `index.js`
So add a file to the main folder

```
$ touch index.js
```

Add the below code to `index.js`

```javascript
const express = require("express");

//Set up express app
const app = express();

app.get("/api", (req, res, next) => {
  console.log("Get Reuest");
  res.send({ name: "Tom" });
});

// Listen for requests
app.listen(process.env.port || 5000, function() {
  console.log("App is listen for requests...");
});
```

Then run this code in the termianl to check if code is working correctly

```
$ node index
```

Once complete install Nodemon for automatic file tracking
Use `--save-dev` to save it to you dev dependencies

```
$ npm install nodemon ---save-dev
```

Now run Nodemon or add `nodemon index` to the npm start script

```
$ nodemon index
```

## Set up your API handling

Create a folder in the main folder called `routes`

```
$ mkdir routes
```

Then add a file into routes called `api.js`

```
$ cd routes
```

```
$ touch api.js
```

Now add the below code to `api.js`

```javascript
const express = require("express");
const router = express.Router();

// Get a list of recipes from the database
router.get("/recipes", (req, res, next) => {
  res.send({ type: "GET" });
});

// Post recipe to the database
router.post("/recipes", (req, res, next) => {
  res.send({ type: "Post" });
});

// Update recipe from the database
router.put("/recipes/:id", (req, res, next) => {
  res.send({ type: "PUT" });
});

// Delete recipe from the database
router.delete("/recipes/:id", (req, res, next) => {
  res.send({ type: "DELETE" });
});

module.exports = router;
```

## Add the `routes/ api.js` file to the main `index.js` file

First off remove from `index.js`

```javascript
app.get("/api", (req, res, next) => {
  console.log("Get Reuest");
  res.send({ name: "Tom" });
});
```

The updated `index.js` file should look like the code below

```javascript
const express = require("express");

//Set up express app
const app = express();

app.use("/api", require("./routes/api"));

// Listen for requests
app.listen(process.env.port || 5000, function() {
  console.log("App is listen for requests...");
});
```

> Next it is recommended to install Postman for testing `API Handing`

## Capturing post data

We need to install middleware.. in this case it's `body-parser`

```
$ npm install body-parser --save
```

Then update your `index.js` file

```javascript
const express = require("express");
const bodyParser = require("body-parser");

//Set up express app
const app = express();

app.use(bodyParser.json());

// The app continues below...
```

From here update the POST request in the `api.js` file

```javascript
router.post("/recipes", (req, res, next) => {
  console.log(req.body);
  res.send({
    type: "Post",
    name: req.body.name,
    price: req.body.price
  });
});
```

## Building out Database Schema

First install mongoose

```
$ npm install mongoose --save
```

Add mongoose `models` to the main folder
Then add `recipe.js` to the models folder

```
$ mkdir models
  cd models
  touch recipe.js
```

Now add the below code to `recipe.js`

```javascript
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

// Create Recipe Schema & model
const RecipeSchema = new Schema({
  name: {
    type: String,
    required: [true, "Name field is required"]
  },
  price: {
    type: Number,
    required: [true, "Name field is required"]
  },
  internal: {
    type: Boolean,
    default: false
  }
});

// The model will represent the name of the database collection
const Recipe = mongoose.model("recipe", RecipeSchema);
module.exports = Recipe;
```

## Connect to mongoDb database

Start off by updating the `index.js` file

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const mongoose = require("mongoose");

//Set up express app
const app = express();

// Connect to mongoDb
mongoose.connect("mongodb://localhost/reciperevenue");
mongoose.Promise = global.Promise;

// The rest of the code in the file continues below..
```

Now update the `api.js` post request

```javascript
// Post recipe to the database
router.post("/recipes", (req, res, next) => {
  Recipe.create(req.body) // This returns a promise while it is processing
    .then(recipe => {
      res.send(recipe);
    });
});
```

Now add connection to the Recipe model in api.js

```
const Recipe = require("../models/recipe");
```

Your database is now connected!!
Head over to Postman and send a post reques to `http://localhost:5000/api/recipes`
The raw json data example needs to match the model format

```javascript
{
	"name": "cake",
	"price": 97,
	"internal": false
}
```

## Error handling

This catches the error and returns the error message to the API for further handling
Update the post script in `api.js`

```javascript
// Post recipe to the database
router.post("/recipes", (req, res, next) => {
  Recipe.create(req.body) // This returns a promise while it is processing
    .then(recipe => {
      res.send(recipe);
    })
    .catch(next);
});
```

As you can see we add a catch method to the end of the promise
This simply calls for the `next` middleware in the express app API
To complete the error handling add this code below to `index.js`

```javascript
// Initialize routes
app.use("/api", require("./routes/api"));

// Error handling middleware
app.use((err, req, res, next) => {
  res.status(422).send({ error: err.message });
});
```

## Delete & Update Recipe

Update the delete & put requests in `api.js`

```javascript
// Update recipe from the database
router.put("/recipes/:id", (req, res, next) => {
  Recipe.findByIdAndUpdate({ _id: req.params.id }, req.body).then(recipe => {
    Recipe.findOne({ _id: req.params.id }).then(recipe => {
      res.send(recipe);
    });
  });
});

// Delete recipe from the database
router.delete("/recipes/:id", (req, res, next) => {
  Recipe.findByIdAndRemove({ _id: req.params.id }).then(recipe => {
    res.send(recipe);
  });
});
```

## Add on the front end

We are going to use React
First install it..

```
$ npm install create-react-app -g
```

Then add react to your project from the main folder

```
$ create-react-app client
```

Once complete update the `index.js` file to point to the client folder

```javascript
// Connect to mongoDb
mongoose.connect("mongodb://localhost/reciperevenue");
mongoose.Promise = global.Promise;

//add in this section
app.use(express.static("client/public"));

app.use(bodyParser.json());
```

## Get React to work

We need to run a script from the `client folder to get React working in the browser
This means we will have two servers running

> React: http://localhost:3000/
> Express: http://localhost:5000/

```
$ npm start
```

Once complete open `package.json` in the client folder
Tell the react server to use a proxy server 5000
Add this line of code underneath the scripts section

```javascript
    },
    "proxy": "http://localhost:5000"
  }
```

## Sync form with React on the client side

Lastly add this code to `app.js` in the src folder in the client folder

```javascript
import React, { Component } from "react";
import "./App.css";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { recipes: [] };

    // this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(e) {
    e.preventDefault();
    // const recipeName = e.target.name.value;
    fetch("/api/recipes")
      .then(data => {
        console.log(data);
        return data.json();
      })
      .then(json => {
        this.setState({
          recipes: json
        });
      });
  }

  render() {
    var recipes = this.state.recipes;
    console.log(recipes);
    recipes = recipes.map((recipe, index) => {
      return <li key={index}>{recipe.name}</li>;
    });
    return (
      <div className="App">
        <form id="search" onSubmit={this.handleSubmit}>
          <label>
            Enter Recipe:{" "}
            <input
              type="text"
              name="name"
              placeholder="Recipe Name?"
              required
            />
          </label>
          <input type="submit" value="Find Recipe" />
        </form>
        <ul>{recipes}</ul>
      </div>
    );
  }
}

export default App;
```

## You should now have a basic app to build from..

Now the fun begins,
Tom