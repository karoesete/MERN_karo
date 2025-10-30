
# MERN STACK IMPLEMENTATION

## IMPLE TO-DO APPLICATION ON MERN WEB STACK

## SIMPLE TO-DO APPLICATION ON MERN WEB STACK

*Task of the project to implement a web solution based MERN stack in AWS cloud.*

- Signed into my AWS account and launched an EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)
- Connected to my EC2 instance through MobaXterm SSH client.

*Backend configuration*

- update ubuntu by running command `sudo apt update`
- upgrade ubuntu by running command `sudo apt upgrade`
- Get location of Node.js from Ubuntu repositories `curl -sL <https://deb.nodesource.com/setup_12.x> | sudo -E bash -`
- Install Node.js with command `sudo apt-get install -y nodejs`
- Verify node installation with command `node -v` and `npm -v`
you should get the verions back like `v12.22.7` and `6.14.15`

*Application Code Setup*

- Create a new directory for your To-Do project  `mkdir Todo`
- Confirm the Todo directory with ls command and change current directory to the new created on `cd Todo`
- Run the command `npm init` to initialise project, thus creating file name package.json

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.
see `npm help init` for definitive documentation on these fields and exactly what they do.

Use `npm install <pkg>` afterwards to installa package and save it as a dependency in the package. json file.

Press ^C at any time to quit.
package name: (todo)
version: (1.0.0) 
description: A To-Do Application
entry point: (index.js) 
test command:
git repository: 
keywords: todo app
author: Karo
license: (ISC)
About to write to /home/ubuntu/Todo/package.json:

{
  "name": "todo",
  "version": "1.0.0",
  "description": "Create a Todo-app",
  "main": "index. js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }  
  "keywords": [
  "todo",
  "app"
  ],
  "author": "Karo",
  "license": "ISC"
}
Is this OK? (yes) yes
```

*ExpressJs Installation*

- Install with command `npm install express` and create a file with command `index.js`.
- Install dotenv using command `npm install dotenv`.
- Open the index.js file with command `vim index.js` and paste code showing in the screenshot below and save in vim

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

// Middleware to set CORS headers
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

// Default route
app.use((req, res, next) => {
  res.send('Welcome to Express');
});

// Start server
app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

```

- Start server to see if it works opening the terminal `node index.js`.
- To confirm its working the terminal should show **Server running on port 5000**
- You should see `Server running on port 5000`
- Open the EC2 Security group port and create a new inbound rule to open TCP port 5000
- Open browser and try accessing the server's public IP http://<public_ipaddress>:5000/
- You should see the `Welcome to Express` in the browser

*Routes Creation*

- Create folder in the To-do app with command `mkdir routes`.
- Change directory to routes folder with command `cd routes` and create a file api.js with command `touch api.js`.
- Open the file with vim api.js and copy the code showing in the screenshot below.
```
const express = require('express');
const router = express.Router();

// GET all todos
router.get('/todos', (req, res, next) => {
  res.send('Get all todos');
});

// POST a new todo
router.post('/todos', (req, res, next) => {
  res.send('Add a new todo');
});

// DELETE a todo by ID
router.delete('/todos/:id', (req, res, next) => {
  res.send(`Delete todo with ID ${req.params.id}`);
});

module.exports = router;

```
*Models*

- The app is going to make use of Mongodb which is a NOSQL database, thus we create a model.
- The model will be used to define database schema.
- To create a Schema and a model, we will use mongoose which is a Node.js package.
- Return back to the Todo folder and install mongoose `npm install mongoose`.
- Create a new folder `mkdir models` and change directory to newly created models folder `cd models`.
- Create a file inside the model folder `touch todo.js`.
- Open the todo.js file `vim todo.js` and paste the code showing in the screenshot below.
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

// Create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
- Return back to the routes directory and open api.js `vim api.js`, delete the code inside `:%d`.
- Paste the code showing in the screenshot below in the api.js file.
```
const express = require('express');
const router = express.Router();
const Todo = require('../models/todo');

// GET all todos
router.get('/todos', (req, res, next) => {
  // Return all todos, exposing only the 'id' and 'action' fields
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});

// POST a new todo
router.post('/todos', (req, res, next) => {
  if (req.body.action) {
    Todo.create(req.body)
      .then(data => res.json(data))
      .catch(next);
  } else {
    res.json({ error: 'The input field is empty' });
  }
});

// DELETE a todo by ID
router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({ _id: req.params.id })
    .then(data => res.json(data))
    .catch(next);
});

module.exports = router;
```
*MongoDB Database*

- A database is needed to store all our data.
- MongoDB database provided by mLab will be used for storing data.
- Sign up for shared cluster free account ideal for our usecase.
- Change time of deleting entry from 6 hour to 1 week.
- Create a MongoDB database and collection inside mLab
- Create a file in Todo directory `touch .env`.
- Open the file `vi.env` and paste the following

`DB = mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority`

- username, password, dbname and network address should be inputted accordingly.
- We update the `index.js` to reflect the use of `.env`, so that Node.js can connect to the database.
- Simply delete the existing content in `index.js`.
- Open the file `vim index.js`, delete content `:%d`.
- Paste in the code showing in the screenshot below.

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('✅ Database connected successfully'))
  .catch(err => console.log('❌ Database connection error:', err));

// Override deprecated mongoose promise with native promise
mongoose.Promise = global.Promise;

// Middleware
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use(bodyParser.json());
app.use('/api', routes);

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err);
  res.status(422).send({ error: err.message });
});

// Start server
app.listen(port, () => {
  console.log(`🚀 Server running on port ${port}`);
});
```
- Start the server using the command `node index.js`.
- A message displaying "Database connected successfully" will appear.
- if you encounter `internal/modules/cjs/loader:1386 throw err`, this is because Node.js cannot find the file index.js in the current directory you are running the command from.
- To fix it, just make sure you are running node index.js from the directory that contains the index.js file.
- Also, make sure your `.env` file has this format of string in it and make sure you input your correct mongodb username and password
```
PORT=5000
DB=mongodb+srv://esetekaro_db_user:YOUR_PASSWORD@cluster0.yinhdht.mongodb.net/todo_app?retryWrites=true&w=majority
```
- If you get a `MongooseServerSelectionError: Could not connect to any servers... IP isn't whitelisted.` this means MongoDB Atlas requires you to explicitly allow any IP address that conencts to your cluster.
-- Login to Mongodb and open your project Cluster 0
-- Locate Networks Access and then click list IP addresses
-- Click on add IP address
-- You can either add you EC2 IP addresss to the whitelist or allow all IP by inputing `0.0.0.0/0`
-- Save changes
-- Restart server `node index.js`
-- You should now see:
```
🚀 Server running on port 5000
✅ Database connected successfully
```
