
# MERN STACK IMPLEMENTATION

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
**Test Backend code without Frontend using Restful API**

You shall see a message 'Database connected successfully, if so - we have our backend configured. Now we are going to test it. Testing Backend Code without Frontend using RESTful API So far we have written backend part of our to-do application, and configured a database, but we do not have a frontend Ul yet. We need ReactS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code. 

In this project, we will use Postman to test our API. Click Install Postman to download and install postman on your machine. 

Click [**HERE**](https://youtu.be/UGZsOlNiVCI?si=OQV48aBxVbXV7iL7) to learn how perform [**CRUD operations on Postman**](https://medium.com/@austejamitz/postman-test-crud-operations-of-a-rest-api-1716bca21a62) You should test all the API endpoints and make sure they are working. For the endpoints that require body, you should send JSON back with the necessary fields since it's what we setup in our code. 

Now open your Postman, create a POST request to the API http://«PublicIP.or-PublicONS>:5000/api/todos. 

This request sends a new task to our To-Do list so the application could store it in the database.

NOTE: Your headers is marked content-Type as application/json

- Postman will be used to test our API.
- Open Postman to create POST request to the API using [http://Your-IP:5000/api/todos](http://18.216.223.201:5000/api/todos)
- Set Key to Content-Type and Value to application/json
- This will return status: 200 Ok, if it was successfully posted.
- Create a GET request to your API using http://18.216.223.201:5000/api/todos, which displays all existing records from the database.
- Do same for DELETE request - To delete a task - you need to send its ID as a part of DELETE request.

**Frontend Creation**

- Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API.
- We will use the create-react-app command npx create-react-app client to scaffold the app.
- start your instance on new tab, in the Todo directory to create react app by running the code
- In the Todo directory, run `npx create-react-app client`.
- Create a folder in the Todo directory called client where all react code will be added.


**RUN REACT APP**

- Before testing the react app, some dependencies needs to be installed.
- Install concurrently `npm install concurrently --save-dev`. It is used to run more than one command simultaneously from the same terminal window.
- Install nodemon `npm install nodemon --save-dev`. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
- In Todo folder, open the package.json file, the section containing "scripts":{..} should be replaced with the code, 
- list and see first then
- `vim package.json`

```
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```
- Configure proxy on package.json
- Change directory to 'client' directory and open package.json `vi package.json`.
- Add the key value pair "proxy":  `"<http://localhost:5000>"` to the package.json file. cat the file `cat package.json` to see
- The purpose of adding the proxy configuration is to make it possible to access the application directly from the browser using [http://localhost:5000](http://localhost:5000/) rather than the including the entire path like http://localhost:5000/api/todos.
- change directory to the Todo directory and run `npm run dev` .
- To be able to access the application from the internet, open TCP port 3000 on EC2 by adding a new security group.
- To confirm if address is working, run [http://Your-IP:3000/]
- You sould see a page showing the react logo

**Creating React Components**

- Advantage of react is that it makes use of components, which are reusable and also makes code modular.
- There will be two stateful components and one stateless component.
- From the Todo directory run `cd client` and move to src folder  `cd src`, create another folder called components `mkdir components`.
- Move into components folder `cd components`.
- Inside the components directory create three files `touch files Input.js ListTodo.js Todo.js`.
- vim Input.js file and paste the code below
- 

```jsx
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ''
  };

  // Handle input change
  handleChange = (e) => {
    this.setState({ action: e.target.value });
  };

  // Handle adding a todo
  addTodo = () => {
    const { action } = this.state;
    if (action.trim()) {
      axios.post('/api/todos', { action })
        .then(res => {
          console.log(res.data); // You can also call a parent callback to update the list
          this.setState({ action: '' }); // Clear input
        })
        .catch(err => console.log(err));
    } else {
      console.log('Input field required');
    }
  };

  render() {
    return (
      <div>
        <input
          type="text"
          value={this.state.action}
          onChange={this.handleChange}
          placeholder="Enter a todo"
        />
        <button onClick={this.addTodo}>Add Todo</button>
      </div>
    );
  }
}

export default Input;

```

- To make use of Axios, which is a Promise based HTTP client for the browser and node.js.
- Move to src folder, to clients folder and install Axios `npm install axios`.
- Go to component directory `cd src/components`.
- Open ListTodo.js `vi ListTodo.js` and paste the following code.

```jsx
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {todos && todos.length > 0 ? (
        todos.map(todo => (
          <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
            {todo.action}
          </li>
        ))
      ) : (
        <li>No todo(s) left</li>
      )}
    </ul>
  );
};

export default ListTodo;

```

- Then in your Todo.js file you write the following code

```jsx
import React, { Component } from 'react';
import axios from 'axios';
import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  };

  // Fetch todos when component mounts
  componentDidMount() {
    this.getTodos();
  }

  // Get all todos from backend
  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if (res.data) {
          this.setState({ todos: res.data });
        }
      })
      .catch(err => console.log(err));
  };

  // Delete a todo by ID
  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if (res.data) {
          this.getTodos(); // Refresh the list after deleting
        }
      })
      .catch(err => console.log(err));
  };

  render() {
    const { todos } = this.state;

    return (
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos} />
        <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
      </div>
    );
  }
}

export default Todo;

```

- Move to the src folder and open App.js and copy and paste the code below.

```jsx
import React from 'react';
import Todo from './components/Todo';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <Todo />
    </div>
  );
};

export default App;

```

- In the src directory open the App.css `vi App.css` and paste the following code.

```jsx
.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 60%;
  margin-left: auto;
  margin-right: auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5rem;
  color: #787a80;
}

input:focus {
  outline: none;
}

button {
  width: 25%;
  height: 45px;
  border: none;
  margin-left: 10px;
  font-size: 25px;
  background: #101113;
  border-radius: 5px;
  color: #787a80;
  cursor: pointer;
}

button:focus {
  outline: none;
}

ul {
  list-style: none;
  text-align: left;
  padding: 15px;
  background: #171a1f;
  border-radius: 5px;
}

li {
  padding: 15px;
  font-size: 1.5rem;
  margin-bottom: 15px;
  background: #282c34;
  border-radius: 5px;
  overflow-wrap: break-word;
  cursor: pointer;
}

@media only screen and (min-width: 300px) {
  .App {
    width: 80%;
  }

  input {
    width: 100%;
  }

  button {
    width: 100%;
    margin-top: 15px;
    margin-left: 0;
  }
}

@media only screen and (min-width: 640px) {
  .App {
    width: 60%;
  }

  input {
    width: 50%;
  }

  button {
    width: 30%;
    margin-left: 10px;
    margin-top: 0;
  }
}

```

- In the src directory open the index.css `vim index.css` and copy and paste the code below:

```jsx
body {
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", "Ubuntu", "Cantarell", "Fira Sans", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282034;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
}

```

- Go to the Todo directory and run `npm run dev`.
- Assuming there are no error , it should work properly.