## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS
The MEAN stack is a JavaScript-based framework for developing web applications. It is a combination of ;
- MongoDB (Document database) – Stores and allows to retrieve data.
- Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
- Angular (Front-end application framework) – Handles Client and Server Requests
- Node.js (JavaScript runtime environment) – Accepts requests and displays results to end use

### TASK : Implement a simple Book Register web form using MEAN stack.

### I provisioned an AWS EC2 instance with Ubuntu AMI version 20.04. This is a point to note as other versions of Ubuntu might have differing installation process for certain technology stack component,e.g MongoDB

## Step 1: INSTALL NODEJS

* Update and upgrade the ubuntu; ` sudo apt update ` and ` sudo apt upgrade `

* Add certificates; 

``` 
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -


```

* Install NodeJS
` sudo apt install -y nodejs `
- This command installs both nodejs and npm, a node package manager

![1](https://user-images.githubusercontent.com/114196715/195963599-46961c16-9a76-4eb4-93f4-adb000c63527.png)

## Step 2: INSTALL MONGODB
- MongoDB stores data in flexible, JSON-like documents. Here, we shall add book records to MongoDB that contain book name, isbn number, author, and number of pages.

* Import the public key

- Run: ` sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6 `

* Create a list file for Ubuntu 20.04

- Run: ` echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list `

![2](https://user-images.githubusercontent.com/114196715/195963621-4029eebb-0c32-430b-9fb9-bce7a3fca2d7.png)

* NB: I discovered that the above public key and the list file specified only works with Ubuntu v20.04. The default AWS Ubuntu AMI comes with v22.04. As such, surf the web for corresponding public key and list file for installing mongoDB on Ubuntu v22.04 if you want to use that version.*

* Install MongoDB : ` sudo apt install -y mongodb `

* Start the server: ` sudo service mongodb start `

* Verify that the server is up and running: ` sudo systemctl status mongodb `

![3](https://user-images.githubusercontent.com/114196715/195963646-54a05da1-b0ff-4830-bb2b-ee02bc2a8d39.png)

* Install Body-Parser package.

- This is a npm module used to process data sent in an HTTP request body. It provides four express middleware for parsing JSON, Text, URL-encoded, and raw data sets over an HTTP request body.

` sudo npm install body-parser `

* Create a directory named 'Books' and cd into it: ` mkdir Books && cd books `

* In the Books directory, initialize npm project: ` npm init `

* In the 'Books' directory, create a file,'server.js' and input therein the code below using ` vi server.js `:

```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});

```

## Step 3: INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER

- Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. Here, We will use Express to pass book information to and from our MongoDB database.

- We will also use Mongoose package,which provides a straight-forward, schema-based solution to model application data. It will be used to establish a schema for the database to store data of our book register.
 
` sudo npm install express mongoose `

* In the 'Books' directory, create another folder, 'apps' and cd into it: ` mkdir apps && cd apps `

* In the 'apps' folder, create a file named 'routes.js' using ` vi routes.js ` and input therein the code below:

```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};

```

* In the apps folder, create another subdirectory named 'models' and cd into it: ` mkdir models && cd models `

* In the models folder, create a file named 'book.js' using your text editor and input the code below:

```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

```

## STEP 4 – ACCESS THE ROUTES WITH ANGULARJS

- AngularJS provides a web framework for creating dynamic views in our web applications. Here, we will use AngularJS to connect our web page with Express and perform actions on our book register.

* Back in the 'Books' directory, create a folder named 'public' and cd into it : `mkdir public && cd public`. Therein, add a file named script.js using ` vi script.js ` and input the code below:

```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

```

* In 'public' folder, create an index.html file and input the following code into it:

```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

```

* Change back to the 'Books' directory and Start the server by running this command: ` node server.js `

* Now, access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

![Final](https://user-images.githubusercontent.com/114196715/195963566-112d9b94-0103-419a-810f-a118eaf8a64f.png)
