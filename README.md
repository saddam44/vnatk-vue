## VNATK
VNATK is a set of client and server frameworks to get your work started ASAP. Also we all know how tesdius it is to do maintenance of any project specially when client/company is ever changing their requirements. By looking at those issue I was missing somehting that really is fast.

So, here we are with two sets of frameworks, this VNATK-VUE and VNATK-EXPRESS-SEQULIZE.

Both repository contains same README file so pls follow proper link to go to respective repository.

Link to VUEATK-VUE 

Link to VUEATK-EXPRESS-SEQUELIZE 
# VNATK-VUE (FrontEnd with Vue, Vuetify)
VNATKVUE is Frontend part of VNATK (Vue Node AgileTool Kit). Making app development easy by including some best practices as integral part, yet, giving you total customization options.

Dependencies: Vue (2.\*), Vuetify(2.\*), vue-form-base

# VNATK-EXPRESS-SEQULIZE (Backend with Express and Squelize)
This express middleware will give all the fule required from server to this Frontend Repo. 

## Getting started

For simplicity we have our project folder structure as follows

```bash
Your Project Root Folder
 - client # Having vue fronetend
 - server # Having express-sequelize backend
```
In case you are working with microservice architecture, you can have following structure

```bash
Your Project Root Folder
 - VueClientAPP1
 - VueClientAPP2
 - ExpressBackendService1
 - ExpressBackendService2
 - ...
 - ExpressBackendService..N
```

# Step 1.0: setup express app
Considering we are in "Your Project Root Folder"

lets create express app (Server/service) from scratch, you are welcome to use any other way or manual if you know what you are doing

```bash
### FOR NEW SERVICE SETUP

#install express-generator globally, its easy to do setup with this
$yourProjectRoot> npm install -g express-generator
...
$yourProjectRoot>express server --no-view
...
#lets check if a folder with server name created or not
$yourProjectRoot>ls
server

#a default structure is also created in this folder
$yourProjectRoot>ls server
app.js       package.json routes    bin          public

$yourProjectRoot>cd server

#lets install basic things setup by express-generator
$yourProjectRoot>npm install

#install our dependencies now
$yourProjectRoot>npm install --save bcrypt body-parser cookie-parser express-handlebars jsonwebtoken morgan cors dotenv lodash mysql2 sequelize vnatk-express-sequelize

### If required vnatk-express-sequelize can be installed in existing express seuelize setup also with very ease

```
## Step 1.1: setup configurations

open ```config/config.json``` file and replace ```operatorsAliases``` value from false to the follwoing
```
operatorsAliases: { $lt: Op.lt, $gt: Op.gt, $like: Op.like },
```

## Step 1.2: configure and use vnatk-express-sequelize

Please add the following code in your app.js 
```javascript
// somewhere on the top after 
// var express = require('express'); <= after this line
const bodyParser = require('body-parser');
const vnatk = require('vnatk-express-sequelize');
...
...
// You can already have body-parser added, no need to add again
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: true }));


const Models = require('./models');
app.use('/crud', vnatk({ // "/crud" will be your base path where the system will hit for its APIs
    Models: Models,
    router: express.Router()
}));
```
### Step 1.3: setup Models

Create models in ```models``` folder

Let's have a sample model. Please read Model comments as documentation


```javascript
'use strict';
const { Model } = require('sequelize');

module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    static associate(models) {
      // define association here
      User.belongsTo(models.City, { foreignKey: 'city_id' });
      User.belongsTo(models.State, { foreignKey: 'state_id' });

    }
    // you have to create this STATIC functionin your model to tell what type of actions are doable on any model
    static vnAtkGetActions(models) {
      return [
        {
          // any identifier of action
          name: "deactivate",
          /* type defined at what level a action will be applicable
          'single'=> for Actions that are executable on any loaded row like activate, or deactivate.
          'NoRecords => for Actions that are doable at Model level ie sendEventInvitationToAll
          'MultiRecords' => for Actions that are doable on selected records by selecting checkboxes in data-grids ie submitSelected
          */
          type: 'single',
          // Your data-grid may have many records but if you want that action to be available only on specific conditions
          // put your where, make sure you have those columns available in model->attributes if defined ie status in bellow where
          where: {
            status: 'Active'
          },
          // actual function in this model to be called, created bellow
          execute: 'deActivate'
        },
        {
          name: "activate",
          type: 'single',
          // If you want a different caption on DataGrid or Forms for your field
          caption: 'Activate',
          where: {
            // Multiple conditions, Array is considered like OR
            // Multiple fields will be in AND Condition
            status: ['InActive', 'Blocked']
          },
          execute: 'activate'
        },
        {
          name: "block",
          type: 'single',
          caption: 'Block',
          where: {
            status: ['InActive', 'Active']
          },
          execute: 'block',
          /**
           * if defined "formschema" the action becoms UI based action and on click
           * will generate a form with proper validations etc. On Submitting this form
           * your "execute" value will be executed on model with sending all records of row along with these form values.
           * "EVERYTHING IS OVERRIDABLE AND CUSTOMIZABLE"
           * This is important. Even if you define something here, On Frontned you can override formfield, validations etc as per page to page need
           * and still all things will work like a charm.
           * For more about Form Schema please read at 
           * 
           * This is important link below
           * 
           * https://wotamann.github.io/ 
           */
          formschema: {
            reason: {
              type: "text",
              label: "reason",
            },
            password: {
              type: "text",
              ext: 'password',
              label: "Your Password to allow blocking",
              clearable: true,
              // solo: true,
              class: "mx-1 mt-1"
            },
          }
        },
      ]
    }
    // Actual function that will be called on loaded model
    // In other way of sequlize you define instance variable as 
    // sequelize.prototype=function () { ... }
    deActivate() {
      this.status = 'InActive'
      return this.save().then(self => {
        return self;
      })
    }

    activate() {
      this.status = 'Active'
      return this.save().then(self => {
        return self;
      })
    }

    // block action is defined with formschema and will receive that in formdata when submitted
    block(formdata) {
      // console.log(formdata);
      this.status = 'Blocked'
      return this.save().then(self => {
        return self;
      })
    }

  }

  // This init method is defined by 'define' method in other way. technically its same ...

  User.init(
    {
      name: {
        type: DataTypes.STRING,
        // caption is vnatk field property to define its caption at frontend
        caption: 'Name',
      },
      email: {
        validate: { isEmail: true },
        type: DataTypes.STRING,
        defaultValue: 'guest@example.com'
      },
      mobile: {
        type: DataTypes.STRING,
        validate: {
          isNumeric: {
            msg: 'Mobile number can only contains number',
            args: true
          },
        },
      },
      password: {
        type: DataTypes.STRING,
        ext: 'password' // VNATK
      },
      status: DataTypes.ENUM('Applied', 'Active', 'InActive', 'Blocked'),
      createdAt: {
        type: DataTypes.DATE,
        field: 'created_at'
      },
      updatedAt: {
        type: DataTypes.DATE,
        field: 'updated_at'
      }
    },
    {
      sequelize,
      modelName: 'User',
    }
  );
  return User;
};
```

Pretimuch thats it... let's setup Vue frontend now

## Step2: setup Vuetify app



