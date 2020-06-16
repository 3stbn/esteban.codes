---
title: "MEVN Tutorial "
date: 2020-06-09T03:27:07.000Z
draft: false
toc: false
images: null
tags:
  - vue
  - javascript
  - node
  - mongo
thumbnail: /images/uploads/thumbnail.png
---
Being able to build, and deploy full-stack applications is a crucial developer skill nowadays and today we are going to learn how to do so using modern web technologies like vue js wich is a popular javascript front-end framework, node js along express for creating a rest APIs and Mongo DB with is the most popular no-SQL database out there. What is usually known by the name of the MEVN stack. By the end of the tutorial we will deploy the app to the internet so you can share this project to your friends.

The app consist in an interface to create bucket list items , so you can keep track of everything that you want to do before dying. 

```html
🗼 Visit Tokyo
🎾 Learn to Juggle
☃️ Sleep in a igloo
```

We can remove the items that we don't longer want , and we can edit current values .

As you can see we are using a live url in the internet, and every request is being send to our backend node server.

We will start from scratch and I will explain every step involve in this process

---

Let's start by setting up MongoDB, we are going to be using mongo atlas which is a service to store mongo in the cloud. you could also set up mongo locally for the purpose of this tutorial. First sign up for an account, once you are done select the free plan that gives you a sandbox to play around with a mongo cluster. We are going to select the default settings here for the region and the type of cluster. You could change the name of it lets leave it by the default. 

Once you save the settings,  lets wait for a few minutes since it can take a lit bit of time to create the cluster.  When the process is completed let's click on the connect button which will give us a quick way to add users and whitelist IP addresses for connecting. 

Let's start with the user, you want to make sure that you use a strong password and save this for later since we will use this in our application.

The IP address that we are going to set up for this tutorial will give access to anyone that has your user password. Ideally, this IP address should be the one that you have your server in production, but let's keep it open for the tutorial.

```bash
0.0.0.0/0
```

When you have added that, in the next step select this option that will give you the URI that we will be using to connect to the cluster, replace the password, and save this config for later when we set up our backend.

---

Now moving on with the project lets create the folder that we will be working on. Let's give it a name and then cd into the project

```bash

mkdir bucket-list-mevn
cd bucket-list-mevn
npm init —yes
```

An initialize our package.json file with npm.  At this point make sure that you have installed nodejs in your system, you can download it from the official website to mac, Linux or windows. 

Now let's open the project with vs code. we are going to install some of the dependencies needed for this project, we will start with the backend. 

```bash
npm install express mongoose body-parser cors morgan
```

Express will be our backend framework to handle the request with node, mongoosewill give us tools to connect and interact with Mongo DB, body-parser will transform the body of the requests to json,  cors will allow our front-end dev server to make ajax calls to the backend and morgan will log all the incoming request to the server

```bash
npm install --save-dev nodemon concurrently
```

For the dev dependencies of our project we are going to be using `nodemon` which is a tool to keep our application running and refreshing every time we made a change, and concurrently which allow us to run two commands at the same time (we will be using this to run our backend server and our front-end dev server at the same time)

For the next part lets create a new file called `server.js` which will be our entry point for the application/. Lets quickly make a hello world app to see if everything is working as expected. Let's require express and declare our app and the port that we want to listen

```js
const express = require('express')
const PORT = 3000
const app = express()
```

Now lets declare this get method from the app that  will return a hello world message to anyone requesting access to the root of our server 

```js
app.get('/', (req,res) => res.send('Hello world'))
```

Finally lest listen to that port  and log to the console if the operation was successful

```js
app.listen(PORT, () => console.log(`App listening at http://localhost:${PORT}`))
```

Lets open the terminal and execute our program

```bash
node server.js
```

For the next part we are going to be adding adding some scripts to run our server, and change the main property to server.js

```json
"main": "server.js",
"server": "nodemon server.js --ignore 'client/'",
```

This will refresh our application every time we make a change and we are passing the ignore flag to ignore our client vue app that we are going to set up now

Lets create a new app with the `vue-cli` first make sure that you have installed it with

```bash
npm install -g @vue/cli
```

And now lets create our vue app with this comand

```bash
vue create client -m npm
```

Because I have yarn installed in my computer I am going to be using the -m flag which will tell the cli to use npm to install the packages 

Here lets select the manually option and unselect the linter 

It should take a minute to install everything now let's take a look at what we have in our package.json, we can see that we have a serve script that we can use to run the front-end app with hot reload. What we want to do now is being able to run the dev script from the server and the one from the client at the same time, of course, you can run one a time but it will be more convenient to have both running with one command.

Lets go to the package.json of the server and there we will add the following script

```bash
"client": "npm run serve --prefix client",
```

What this command will do is run our the dev server of the client folder in this case our vue app, and now lets add this dev script .

```bash
"dev": "concurrently \"npm run server\" \"npm run client\""
```

What this will be doing is run our server and our client at the same time

(Testing the script )

And the last thing for this is initial configuration, is that I want to be able to make API calls from the client with the root path of our server, I mean without the need to add [localhost](http://localhost) or another host in the URL. For doing that I am going to set up a proxy config for webpack . Let's create a `vue.config.js` file and add the following config

```js
module.exports = {
    configureWebpack: {
        devServer: {
            proxy: {
                '/api': {
                    target: 'http://localhost:3000',
                },
            },
        },
    },
}
```

We need this only for development so we dont have to add the url in our ajax requests from the frontend, and every-time we try to access the api path we will proxy the request to localhost:3000.

Now that we have our starter project lets begin with our backend api.

We will start by adding a config file ,  we can use this file  to connect to the correct port and to mongodb , in the root of the project create a file called `config.js` and add the following

```js
module.exports = {
    mongoUri: process.env.MONGO_URI,
    PORT: process.env.PORT || 3000,
}
```

We are exporting a config object that has the mongoURI that will be using to connect to our mongo cluster, this will be the one that we configured in mongo atlas. 

I am going to store this in an enviroment variable because I don't want to check this sensitive information into GitHub, I advise you to do the same but you could also have the URI in play text here if you don't plan to public this repository .

```js
export MONGO_URI='<your mongo uri>'
```

This will store the mongo URI as an environment variable, that we can read in our code with `process.env.MONGO_URI` be aware that this variable will only be available as long I keep this terminal session open  so for future use you can add this to your bashprofile were you have your other env variables.

For windows you can run the same command but make sure that you are using the `git bash terminal`.

For the port we are also going to use a environment variable that usually is set in our production servers, if we don't have that variable it will fallback to the port 3000.

Now that we have our config lets open the server.js file and lets connect to mongoDB with mongoose

First lets require the package and our config file in 

```js
const mongoose = require('mongoose')
const { PORT, mongoUri } = require('./config')

mongoose
    .connect(mongoUri, {
        useNewUrlParser: true,
        useCreateIndex: true,
        useUnifiedTopology: true,
				useFindAndModify: false
    })
		.then(() => console.log('MongoDB database Connected...'))
    .catch((err) => console.log(err))
```

Because the connection is an asynchronous operation we are going to be using promises and once the DB is connected we are going to console log a message, if the operation fails we are logging the error 

Lets run our server to see if this works.

Finally we are going to make use of the middlewares that we installed 

```js
const cors = require('cors')
const morgan = require('morgan')
const bodyParser = require('body-parser')

app.use(cors())
app.use(morgan('tiny'))
app.use(bodyParser.json())
```

Cors will allow us to make ajax request from our frontend dev server to the backend dev server. Morgan will log every request to the server in the console and we are using the 'tiny' config that gives us a small message, and body-parser will transform the body of every request to json

Once we have this set-up lets create a schema and mongo model with mongoose

I am going to create a new folder called models and here we will create a file BucketListItem.js, so this will be the name of our model, lets import Schema and model from mongoose

```js
const { Schema, model } = require('mongoose')
```

And now lets declare our schema

```js
const { Schema, model } = require('mongoose')

const BucketListItemSchema = new Schema({
    description: {
        type: String,
        required: true,
    },
    date: {
        type: Date,
        default: Date.now,
    },
})

```

It will be an object that has a description property which is a required string, and it will also have a option  to add a date that will be the type of the javascript Date object, and the default value will be the the current time of the server.

Now we are going to declare our model with the schema and export it, so it can be used from outside.

```js
const BucketListItem = model('bucketListItem', BucketListItemSchema)
module.exports = BucketListItem
```

Now that we have declared our model lets create the routes for the rest api. I am going to create a folder called `routes` and in here lets create a file called `bucketListItems` in a folder called api

Here first require the modules that we need

```js
const { Router } = require('express')
const BucketListItem = require('../../models/BucketListItem')
const router = Router()
```

We are going to use Router from 'express' and our model that we declared before, now lets declare an instance of the Router

The first route that we are going to the declare is the root of this endpoint 

We are going to use our router get method that can handle get request and for the second argument we can pass an async callback function to handle our logic

Because this is the root of our endpoint we are going to await for all the items in the collection with the find method that we have in our model

If theres no result we are going to throw an error, otherwise we are going to first sort the response by date using the array sort method and then transforming our date object to milliseconds so we can subtract it and compare it.

Finally we are going to send back the our sorted items  response with an status of 200 .

Because this promise could fail lets handle a possible error with a try catch block

If the request fails we are going to send back a 500 with the message of the error.

```js
router.get('/', async (req, res) => {
    try {
        const bucketListItems = await BucketListItem.find()
        if (!bucketListItems) throw new Error('No bucketListItems')
        const sorted = bucketListItems.sort((a, b) => {
            return new Date(a.date).getTime() - new Date(b.date).getTime()
        })
        res.status(200).json(sorted)
    } catch (error) {
        res.status(500).json({ message: error.message })
    }
})
```

Now lets handle a post request to create new items

We are going to use the post method of our router and we are going to listen again to the root of this endpoint, the callback will also be an async function that takes the req and the response

Now lets declare a variable with our bucketlistitem constructure , we will pass our req.body (that has all the properties that we need to store) to the constructure ,  

Lets add another try catch block that will handle our async code, we are going to declare a `bucketListItem` const that will be the result of our model save operation, don't forget to await the result. If we don't have a result we are going to throw a new  error and then we are going to send the object saved with a 200 status, the catch block again will send back a 500 error with the error message.

```js
router.post('/', async (req, res) => {
    const newBucketListItem = new BucketListItem(req.body)
    try {
        const bucketListItem = await newBucketListItem.save()
        if (!bucketListItem) throw new Error('Something went wrong saving the bucketListItem')
        res.status(200).json(bucketListItem)
    } catch (error) {
        res.status(500).json({ message: error.message })
    }
})
```

For updating existing items we are going to use the put method  of our router and we are going to declare the id of the item as a param with this colon syntax.

for our async callback function we are going  to use our try catch block and first lets deconstruct or id from the req.params object, with that lets await for the findByIdAndUpdate method from our model, the first param will be our id and the second is going to be our updated item that comes from the req.body

This method will make sure that this item exist and then it will update it . 

I want to return the updated object to the client for doing that I will spread the response._doc (which is the doc that we found with the method) , with the updated properties that come from the body of the  request

As before we check that we have a response and then we send it back with a status of 200, otherwise we throw an error and then send back an status of 500 with the message of the error.

```js
router.put('/:id', async (req, res) => {
    const { id } = req.params

    try {
        const response = await BucketListItem.findByIdAndUpdate(id, req.body)
        if (!response) throw Error('Something went wrong ')
        const updated = { ...response._doc, ...req.body }
        res.status(200).json(updated)
    } catch (error) {
        res.status(500).json({ message: error.message })
    }
})
```

Finally we need a way to delete items , for doing that we are going to use our delete method from our router and like our put method, we are going to declare the id of the item as a param with this colon syntax.

The async callback function in here will also have a try catch block, we first use the id from the req.params object and then lets await for findByIdAndDelete from our model , this method will take the id as a param and it will return the removed item, if we have the item we  send a 200 response  otherwise we throw an error and then send a 500 response with the message

```js
router.delete('/:id', async (req, res) => {
    const { id } = req.params
    try {
        const removed = await BucketListItem.findByIdAndDelete(id)
        if (!removed) throw Error('Something went wrong ')
        res.status(200).json(removed)
    } catch (error) {
        res.status(500).json({ message: error.message })
    }
})
```

Now lets export our router so it can be used from other files

```js
module.exports = router
```

At this point we have declared all of our routes and models now we only need to use them in the entry point file, lets open server.js and require our routes

```js
const bucketListItemRoutes = require('./routes/api/bucketListItems')
```

Then we are going to use the routes for the `api/bucketListItems` endpoint

```js
app.use('/api/bucketListItems', bucketListItemRoutes)
```

## Vue

Now that we have our server ready lets move on to the vue front end.

First lets open the terminal and cd to the client folder .There we will   intsall axios 

```js
npm install axios
```

Axios will help us to make Ajax request to our server, feel free to use the native fetch api if you want.

Lets cd back to the root of the  folder and run the server and the frontend at the same time

```js
cd ..
npm run dev
```

Lets clear up our code, In the App.vue file i will first remove the initial code from the cli and then I will remove the initial component, now open the index.html file of the public folder and here we are going to add the cdn for bulma, wich is the css framework that we are going to use we will also  add the cdn for material icons

```js

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.8.2/css/bulma.min.css" />
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />
```

The first thing  that we are going to do in our main vue component is fetch all the items from the our server. Lets first declare a new data property that will be an empty array to store all of our items

```js
data() {
    return {
      items: [],
      description: "",
      editedDescription: "",
      selected: {}
    };
},
```

Next lets  use the mounted vue lifecycle hook to make our api call to the get all of the items. This hook will be called when all of your dom elements are rendered in the dom. The  we are going to use the async keyword so we can make use of await

```js
async mounted() {
    const response = await axios.get("api/bucketListItems/");
    this.items = response.data;
},
```

For making the Ajax call we are going to use axios , So lets import axis first 

```js
import axios from 'axios'
```

Now in the mounted function Lets await for the api call with the get method from axios, we will pass the route of our backend api and then lets store the response in a variable, Now lets assign the data of the response to the items data property of the component.

This should take care our fetching our data but now we need to display this data to the dom, lets first add a div with the id "app" and give it some style so it does not expand in all the screen

```js
<div id="app">
</div>
```

```css
#app {
    margin: auto;
    margin-top: 3rem;
    max-width: 700px;
}

```

now for every item we are going to use a div that has the notification css bulma class

```js
<div class="notification" v-for="(item, i) in items" :key="item._id">
  <span class="tag is-primary">{{ i + 1}}</span>
	<p>{{ item.description }}</p>
</div>
```

Lets add the v-for directive in the element and we will read the item and the index from the items array, we will also bind the key prop with the id of every item , this will help vue to keep track of which dom elements are being added and removed from the dom. The children of this element will be a p node and inside we are going to bind the item description with the vue template syntax 

We now should be able to see a dom node for each item of  our backend. 

Now we need a way to add new items with the front -end

Lets first add an input element with the help of bulma css classes

```html
<div class="field has-addons">
    <div class="control is-expanded">
        <input class="input" type="text" placeholder="Go to mars..." />
    </div>
    <div class="control">
        <a class="button is-info">Add</a>
    </div>
</div>
```

 Now we are going to declare a new data property 

```js
description: '',
```

The initial value will be an empty string that we will be binding to our input with v-model, v-model will take care of reading and updating the description property every-time we interact with the element

```js
v-model="description"
```

Now we need a way to save the value of the input to the backend

Lets add a click event listener to our button and pass the addItem function to it. We will also disable the button if there is nothing in the description so we don't have empty values

```js
@click="addItem" :disabled="!description"
```

The addItem method does no exist  yet so lets create it in the methods part of our component

```js
async addItem() {
  const response = await axios.post("api/bucketListItems/", {
    description: this.description
  });
  this.items.push(response.data);
  this.description = "";
},
```

The method will use async await  

Here  we will first await to the to the post request that we are making with axios, the route will be the same as the one from our get call and the data will be an object that has the current description value of our component, once this is completed we are going to be pushing to the items array the data value of the  response which is our item created, after that we need to reset our input to the initial state.  

Now we need a way to remove items from the database , for doing that lets change our item html markup so we can have two columns 

```html

<div class="columns">	
	
</div>

```

```js
<p class="column">{{ item.description }}</p>
```

```js
	<div class="column is-narrow">
  </div>
```

We will create a new div element with the `columns` class provided by bulma and then a column class to each child, the second child will have the is narrow class so it only uses the space that it needs, the second child will have our material icon elemnt  and we will listen to the click event to call the removeItem method with the item and the index as arguments

```html
<span class="icon has-text-info" >
  <i class="material-icons">delete</i>
</span>
```

```html
@click="removeItem(item, i)"
```

The removeItem method will be also async

```js
async removeItem(item, i) {
    await axios.delete("api/bucketListItems/" + item._id);
    this.items.splice(i, 1);
},
```

We will first await for the axios.delete method , the route will be our base url concatenated with the id of the item so the server knows what to delete, after that we are going to remove the item from our items array with the splice method

Lets test this out! you can see that this is removed from the dom , and if we refresh the screen , we can see that the change was persisted in the db.

The only thing that our ui is missing now , is the ability to edit current items . For doing that we are going to be adding a edit icon at the left of the delete icon

```html

<span class="icon has-text-primary">
  <i class="material-icons">edit</i>
</span>

```

To keep track of what we are updating I am going to create two new data properties

```js
editedDescription: "",
```

```

the editedDescription will be a string that gets updated when we edit the item, and the selected object will be the item that we click 

Now lets add a select method that we will invoke when we click in the edit icon of an item.

```js
select(item) {
  this.selected = item;
  this.editedDescription = item.description
},
```

This method will take the item as a parameter and it will  update the selected and editedDescription state with the item values

Now lets add the event listener to the icon  

```html
 @click="select(item)"
```

We now need to display an input box based on the selection

```js
<input class="column input"   v-if="selected._id === item._id" v-model="editedDescription"/>
```

We will use the v-if directive that it will render the element if the selected._id prop is the same as the one from the item , the value that we are going to bind here to v-model is the editedDescription property that we created before to keep track of what we are editing

Don't forget to add the v-else directive to the plain readonly p tag so this will be displayed if the previous condition is false 

```js
v-else class="column"
```

Lets test if this works.

I don't want to write all of this selected logic every time I need to check if the item is selected so lets write a method that can handle this

```js
isSelected(item) {
  return item._id === this.selected._id;
},
```

This method will return a boolean that indicates if a item is selected 

now lets use it in our template

```js
v-if="isSelected(item)"
```

Once we are in edit mode i would like to have a way to go back to the readonly state and a way to save the changes , lets add that 

```js
{{isSelected(item) ? 'close': 'edit'}}
```

I am using a ternary operator here, if the item is selected I am going to show the close icon otherwise it will show the edit icon,

Lets do the same for our click listener 

```js
isSelected(item) ?  unselect() : select(item)
```

If the item is selected we can unselect  otherwise we will select, lets write our unselect method

```js
unselect() {
  this.selected = {};
  this.editedDescription = "";
},
```

This method does not receive any params and it will only set our edit data to the default

We are going to do the same for the trash icon

```js
isSelected(item) ? 'save': 'delete'
```

If the item is selected we will show the save icon otherwise the delete one

```js
isSelected(item) ? updateItem(item, i) : removeItem(item, i)
```

the click event will also have logic, if the item is selected wi will updated other wise we are going to remove it

The updateItem method will look like this

```js
async updateItem(item, i) {
  const response = await axios.put("api/bucketListItems/" + item._id, {
    description: this.editedDescription
  });
  this.items[i] = response.data;
  this.unselect();
}
```

it takes the item and the index as params

we will first await for the put operation with axios and store the result in a const declaration, the route will be our base url concatenated with the item._id param, the data to update, that axios takes a second param, will be the an object with the editedDescription that we keep track in our state

Once that is completed we will update our array with the updated value with the data of the response.

After that we are going to unselect so the user leaves the edit mode.

Lets test this out.

We can see that we can enter the edit mode end then leave it clicking in the close icon

If we want to update we click the save button and the item leaves the edit mode

Lets refresh the page and we can see that our change was persisted to the mongo database.

Finally, lets add this css so the icons cursor is a pointer when we hover in

```css
.icon {
  cursor: pointer;
}
```

And lets add an h1 tag with the title of the site

```js
<h1 class="subtitle has-text-centered">Bucket List:</h1>
<hr />
```

## Deployment

Now that we have everything working lets deploy our app to heroku. 

Heroku is a managed infrastructure platform to deploy applications with that the need to set up servers manually, it offers a free tier that is the one that we are going to use

Before deploying we need to add some scripts to the package.json of the server , this scripts will be executed when we deploy our app. The first script will run our application when deploying, heroku requires this start script

```json
"start": "node server.js",
"build": "npm install --prefix client ",
```

The next one will build our client from the root of our app, we first need to install the dependencies of the client and then build the vue app with the build command prefixed for the client folder

```js
&& npm run build --prefix client
```

This command will run when we push our code to heroku.

The last change that we will be doing will be on the server.js file, here we need to serve our static files when we are in prod, this will be our builded vue application 

Lets remove our previous hello world message

```js
if (process.env.NODE_ENV === 'production') {
    app.use(express.static('client/dist'))
    app.get('*', (req, res) => {
        res.sendFile(path.resolve(__dirname, 'client', 'dist', 'index.html'))
    })
}
```

We will first check if the node env is equals to production, this will only be true when the app is running in the server or if we change or node env variable,

if the app is running in production we will make use of the static method from express and what we will serve is the client public folder what has our static files after the build

Now for any route that the user tries to access we will respond with the index.html file of the client public folder

we are making use of the node path function so make sure to require it 

```js
const path = require('path')
```

Moving on with the deployment. Sign up for an account in heroku if you haven't and lets create a new app, give  it the name to you want  i will name it mevn-bucket-list-app. We will need the heroku cli so go to the download page to install it with the correct method for you operating system. Because I am in mac I prefer to used the brew method.

```bash
heroku --version 
```

We will now verify that the cli tool is installed.

Now lets run heroku login to authenticate with the cli

```bash
heroku login
```

Heroku uses git to push your code to the server, lets initialize our git repository with git init (and make sure that you have git in your system)

```bash
git init
```

Now we will ad a .gitignore file to the root of the project because we don't want to check our node_modules file to the source control

```bash
node_modules
```

We can now stash all of our files to git and then commit 

```bash
git add .
git commit -m "first commit"
```

Now lets add our remote git origin to heroku with:

```bash
heroku git:remote -a <your heroku app name>
```

before pushing our code to the server we need one more thing to setup, as you remember we stored our mongo url as an env variable we need save this env variable in the heroku server, we will use the heroku cli for doing this.

 make sure that you have quotes for the variable.

```bash
heroku config:set MONGO_URI='<Your mongo uri>'
```

And because we are building our vue front end with dev dependencies we also need to set the NPM_CONFIG_PRODUCTION env var to false so it can install the dev dependencies and build our frontend

```bash
heroku config:set NPM_CONFIG_PRODUCTION=false
```

This all our the env variables that we need to deploy, you could also check in the console of heroku that the variables were created

Now we only need to push our code to the heroku origin , wi will use the master branch

```bash
git push heroku master
```

This will trigger the build

We can see how all of our dependencies are being installed then the build script is executed to build our vue app and finally it will start the server with the start script that we specified .