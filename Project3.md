## PROJECT THREE: DEPLOYING A MERN STACK

## MERN is one of several variations of the MEAN stack (MongoDB Express Angular Node), where the traditional Angular.js front-end framework is replaced with React.js.
MERN is acronym for; 
- Mongo DB (a document based non-relational database used to store application data in a form of documents.)
- ExpressJS (A server side Web Application framework for Node.js.)
- ReactJS (A frontend framework;client-side, developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components)
- Node.JS (A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser)

![Architecture](https://user-images.githubusercontent.com/114196715/195962630-bfd8d227-5ae3-431c-a276-c7236fbea92c.png)

## STEP 1 – BACKEND CONFIGURATION

* Update and upgrade Ubuntu:
 ` sudo apt update `
 ` sudo apt upgrade `

* To get the loation of the node.js software from Ubuntu repositories, run:
` curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - `

* Install Node.JS on the server:
` sudo apt-get install nodejs -y `

- The above command installs both nodejs and NPM. NPM is a package manager for Node and it is used to install Node modules & packages and to manage dependency conflicts. 

* Verify the Node and NPM installation:
```
node -v
npm -v
 
```

### Application Code set-up
* Create a new directory for your Todo project and cd into it:
` mkdir Todo && cd Todo `

* Initialise your project with the `npm init` command. A package.json file is created which contains information about the application and dependencies needed for it to run.

* Run the command 'ls -l' to check if the package.json file was created.
` ls -l `

### INSTALL EXPRESSJS
Express is a framework for Node.js. It helps to define routes of your application based on HTTP methods and URLs.

* To use express, install it using npm:

` npm install express `

* Create a file named index.js and run ls to confirm that your file was successfully created; ` touch index.js && ls -l `

* Install the *dotenv* module; ` npm install dotenv `

* Open up the index.js file, input the code below and save: ` vim index.js `

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
- Open your terminal in the same directory as your index.js file and type: ` node index.js `

![Database_running](https://user-images.githubusercontent.com/114196715/195962731-6cd585f5-3545-4e88-a067-9ada0605ed7b.png)

* Edit your ec2 inbound rule to allow traffic from port 5000.

* Open up your browser and try to access your server's public IP address;
` http://<PublicIP-or-PublicDNS>:5000 `

![Express](https://user-images.githubusercontent.com/114196715/195962784-1204be3d-8127-425d-b2f0-746c897dcbd6.png)

### ROUTES
There are three actions that our To-do appliation should be able to do:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

- Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.
- For each task, we need to create routes that will define the various endpoints that the todo app will depend on:
- Create a directory named 'routes': ` mkdir routes `

Tip: You can open multiple shells in Putty or Linux/Mac to connect to the same EC2

* Change directory to the routes folder and create a file called 'api.js': ` cd routes && touch api.js `

* Open up the 'api.js' file with ` vim api.js ` and input the code below:

```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;

```

### MODELS
*Points to note*
- Since the app is going to make use of the MongoDB database, we need to create a model.
- A model is the heart of JavaScript based applications, and it is what makes it interactive. These models will be used to define the database schema so we can define the fields stored in each MongoDB document.
- In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties.


* To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.
- cd back to the 'To-do' folder and install mongoose: ` cd .. `
` npm install mongoose `

* Create a new folder *'models'*, cd into the newly created folder and create a 'todo.js' file:

 ` mkdir models && cd models && touch todo.js `

* Open the todo.js file with ` vim todo.js ` and input therein the code below:

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;

``` 

* update the  routes from the file api.js in ‘routes’ directory to make use of the new model.
- In Routes directory, open api.js with `vim api.js`, delete the code inside with `:%d` command and paste the code below:

```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;

```

### MONGODB DATABASE
- To store our data, we shall make use of mLab, mLab provides MongoDB database as a service solution (DBaaS).
- Sign up [here](https://www.mongodb.com/atlas-signup-from-mlab) for a shared cluster account, select AWS as the service provider and choose a region close to you.
- Complete a get started checklist as shown in the image below

![Mongo_DB](https://user-images.githubusercontent.com/114196715/195962821-cc351379-3d19-4d69-af86-59b1a4f9bea7.png)

- Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing)
- Change the time of deleting the entry from 6 Hours to 1 Week
- Create a MongoDB database and collection inside mLab
- In the index.js file, we specified process.env to access environment variables, create a file in the Todo directory and name it .env.

` mkdir .env `

* Open up the file with ` vi .env `, Add the connection string to access the database in it, just as below:

` DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority' `
- Update <username>, <password>, <network-address> and <database> according to your setup

* Update the index.js to reflect the use of .env so that Node.js can connect to the database, delete existing content in the file, and update it with the entire code below.

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
NOTE; Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

* Start your server using the command ` node index.js `

- A ‘Database connected successfully’ message indicates that our database was successfully configured.

### Testing Backend Code without Frontend using RESTful API

So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code.
- We will use [Postman](https://www.postman.com/) to test our API
- Download and install Postman on your machine
- Test all the API endpoints and make sure they are working

* Open Postman, create a POST request to the API
- http://<PublicIP-or-PublicDNS>:5000/api/todos. This request sends a new task to our To-Do list so the application could store it in the database.
- Note: make sure your set header key *Content-Type* as *application/json*

![POST](https://user-images.githubusercontent.com/114196715/195962861-bc972815-7168-477f-a195-046f038480ef.png)

* Create a GET request to your API on http://<PublicIP-or-PublicDNS>:5000/api/todos. This request retrieves all existing records from our To-do application (backend requests these records from the database and sends it us back as a response to GET request).

![GET](https://user-images.githubusercontent.com/114196715/195962885-11d5a08c-a44c-47eb-b6a7-75bd7bd8c35e.png)

## STEP 2 – FRONTEND CREATION
- To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.

* In the same root directory as the backend code, which is the Todo directory, run

`  npx create-react-app client `

This will create a new folder in the Todo directory called *client*, wherein we will add all the react code.

### Running a React app.
- Before testing the react app, there are some dependencies that need to be installed.

1. Install ['Concurrently'](https://www.npmjs.com/package/concurrently), It is used to run more than one command simultaneously from the same terminal window.

` npm install concurrently --save-dev `

2. Install [Nodemon](https://www.npmjs.com/package/nodemon), used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.

` npm install nodemon --save-dev `

3. In Todo folder, open the package.json file. Replace the ''scripts'' and "tests" part with the code below.

```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},

```

### Configure Proxy in package.json

1. Change directory to ‘client’

 ` cd client `

2. Open the package.json file

 ` vi package.json `

3. Add the key value pair in the package.json file "proxy": "http://localhost:5000".

N.B: The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos.

* In the Todo directory,run the command; ` npm run dev `

- The app should open and start running on localhost:3000

* Add a new rule on the ec2 instance and open TCP port 3000.

![REACT](https://user-images.githubusercontent.com/114196715/195962948-899065ed-db9d-40c8-b9bb-9ed06aeebc48.png)

### Creating the React Components

* From the todo directory, run ` cd client `
* Move to the src directory, ` cd src `
* Inside the src folder, create another folder called components and cd into it; ` mkdir components `
* Inside the components directory, create three files 'Input.js', 'ListTodo.js' and 'Todo.js': ` touch Input.js ListTodo.js Todo.js `
* Open up the 'Input.js' folder with ` vi Input.js ` and input the following code;

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input

```


* To make use of Axios, which is a Promise based HTTP client for the browser and node.js, cd into the 'client' folder from the terminal and run `yarn add axios` or `npm install axios`.

* Change direcory into the 'components' folder and open up the 'ListTodo.js' file. Input the code below:

```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo

```

* In the 'Todo.js' file, input the following code:

```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;

```

* We need to make little adjustment to our react code, delete the logo and adjust our 'App.js' file

- Move to the 'src' folder and run ` vi App.js `, input the following code:

```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;

```

* In the src directory, open up the 'App.css' file and input the followin code:

```
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
width: 100%
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

* In the src directory, open up the 'index.css' file and input the following code:

```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}

```

* Move to the todo directory and run ` npm run dev `

![FINAL](https://user-images.githubusercontent.com/114196715/195963013-a450536a-314f-4f9f-88a5-71100320b015.png)
