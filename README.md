# DSPT12: Unit 3 - Sprint 3 Review

## Disclaimer

In efforts to replicate what will be experienced on the Sprint Challenge, all code files will be kept on the same
level in the repository.


In addition, I recommend you **start with a blank repository** before working on this review!


## Premise

You are creating a simple Flask application which will make several calls to Agify.io which, given a person's name, will
return their predicted age.  Your task is to take the JSON response and add the name, age and an ID value to a sqlite3
database with the SQLAlchemy ORM.


For this app, you will have at least three routes which will do the following:

   1. Show all records from the database
   2. Show all records of individuals older than 40
   3. Makes a call to Agify.io and adds the record to a database


## Coding: Part I - A simple Flask application

1. Before you create an application file, create a new virtual environment and install the following packages:

    - Flask
    - Flask-SQLAlchemy
    - requests


2. Next, make a file called `app.py` which will serve as the Flask application file.


3. Make a simple Flask application with the following items:

   - Make your app into a function called `create_app`
   - Name the Flask application `APP`
   - Make your _base_ route and return `'This is the base route.'`
   - At the end of your `create_app` function, return the application variable


4. Now, add a conditional at the bottom of your script which will allow you to start up the Flask app when the script is
run.


5. After starting it up, visit the base route to ensure it is working!


## Coding: Part II - Create the Database

1. In the same application file, make some lines before the `create_app` function to create room for your SQLAlchemy
model.  In class, we put our ORM model into another file but, for simplicity's sake, we can keep them in the same file.


2. After instantiating the database variable from SQLAlchemy, create a table called `Record` which will inherit the
`Model` class from the database variable.


3. Within your `Record` class, create the following attributes (with these data types) all of which are not Nullable:

   - id (Big Integer) - Primary Key
   - name (String)
   - age (Small Integer)

   
4. Lastly, add a `__repr__` method in this class which will emulate the following string representation of your classes:

   - `'[Id: 0 | Name: James | Predicted Age: 27]'`
   

#### ***Back to the application!***

5. In your application function, configure the app's database to the following URI:

   - `sqlite:///agify_data.sqlite3`


6. Afterwards, initialize the application for use with our database setup


#### ***Let's test it out!***

7. We will do something that we did not do during lecture which is work with our app in a _Flask shell_.  To start this
process, we want to set an environment variable for our Flask application.  Accomplish by doing the following in your
terminal:

   - Windows: `set FLASK_APP=app.py`
   - UNIX/Bash/Mac: `export FLASK_APP=app.py`


8. Now that we have set the environment variable, run the shell by entering this command in your terminal: `flask shell`
This will open up a Python REPL (what opens up when you type `python` and enter in your terminal) but, in the context of
our Flask application.


9a. Follow the code listed below - each line is run individually as denoted by started with `>>>`:

```
>>> from app import DB, Record
>>> DB.drop_all()
>>> DB.create_all()
```


9b. What we have done so far is...

   - Imported the DB and Record classes from our app script
   - Dropped the tables/data from our database if it exists
   - (Re)created the schema of our database based on our ORM


10a. Now, create the first record by hand and add it to the database.  It will contain the following attribute values:

   - `id = 0`
   - `name = 'james'`
   - `age = 27`

10b. Check your work!  If you did it correctly, you should be able to run `Record.query.all()` in your Flask shell and
return a list containing the representation of your record: `[Id: 0 | Name: james | Predicted Age: 27]`


11.  Make sure to keep your shell open for the next part!


## Coding: Part III - Test the Agify API

1. Click the link to the (Agify.io API)[https://api.agify.io?name=james] for the name 'james'.  The response should give
us the name, predicted age, and another feature we won't worry about.


2. Now, we will recreate this using our `flask shell`.  Enter the following lines of code into your Flask REPL:

```
>>> import requests
>>> from ast import literal_eval
>>> URL = 'https://api.agify.io/?name=james'
>>> data = literal_eval(requests.get(URL).text)
```


3. Here is what we have done:

   - Imported the `requests` library to make requests to a server
   - Imported the `literal_eval` method from the `ast` library which allows us to convert a "stringified" response into
     its literal representation... ex. `"[0, 1, 2]"` -> `[0, 1, 2]`
   - Hard-coded the URL for the Agify.io API call using the name 'james'
   - Entered that URL into the `requests.get()` method and extracted the text via the `.text` attribute.  This is then
     wrapped in the `literal_eval` method which converts the string representation into its data type.


4. Now, you can extract the following attributes to add to your model (the ID will have to be hard-coded) in the REPL:

```
>>> r0 = Record(id=1, name=data['name'], age=data['age'])  # We are using 1 since we already have an entry with an id=0
>>> DB.session.add(r0)
>>> DB.session.commit()
>>> Record.query.all()
 --> [[Id: 0 | Name: james | Predicted Age: 27], [Id: 1 | Name: james | Predicted Age: 58]]
```


5. Lastly, we will replicate this code within our script!


6. Go back to your `app.py` and create a route called `'/refresh'` which performs the following actions:

   - Drops the tables/data form your database
   - (Re)creates the tables according to your ORM
   - returns a string that says `'Database Refreshed!'`
   - _We will hit this route BEFORE making any calls to the API as to not receive an integrity error in adding the same
      ID twice_


5a. Create another route called `'/check_name''` which will have the following variables to start:

   - `BASE_URL` -> the URL of the Agify.io API with query param *without* the name value
   - `name`     -> extract the query param called `name` out of the URL (`/check_name?name=<name>`)
   - `data`     -> the literal representation of the text received from using the `request.get()` method (aka: what we 
                   did in the Flask shell)


5b. Next, you need to create an if:else conditional that will set the id's value to the max id plus 1, otherwise, start
at `-1`.  This will be tough since we didn't go over this before (this will not be on Sprint Challenge).  You can make a
query to the database by using the `DB.session.query()` method and passing in SQLAlchemy's version of the `max()`
function to grab the max `Record.id` and chaining the `.first()` method after rather than `.all()`.  Then, isolate the
value using tuple indexing (similar to list indexing). 

   - _We will start at `-1` since we will be adding 1 to it when making our Record instance_


5c. Create a try:except statement which does the following:

   - try --> adding a record to the database and returning a string that says `'Record added: '` followed by the string
             representation of the instance.
   - except --> an Exception error as `e` which will then return `e` as a "stringified" response.  _We need to
                "stringify" the response because you can only render certain data types on a page._


6. Now, we will modify our base route to return all the records from out database.  Keep in mind the italicized
   statement above!


7. Lastly, create one more route which will do the same as the previous task but only return records where the person's
   age is 40 or younger.  Call this route `'/no_older_than_40'`.


# Congratulations!

You've finished the review - this will be pretty close to what you will see on the Sprint Challenge so, if you felt like
it was easy, the Sprint Challenge will most likely also be easy.  Good luck!