# Clinical Laboratory Management Web Application

## Table of Contents

1. [Video DEMO](#video-demo)
2. [Description](#description)
   1. [`static`](#static)
      1. [`favicon.ico`](#faviconico)
      2. [`logo.png`](#logopng)
      3. [`styles.css`](#stylescss)
      4. [`transparent.png`](#transparentpng)
   2. [`templates`](#templates)
      1. [`layout.html`](#layouthtml)
      2. [`index.html`](#indexhtml)
      3. [`validacion.html`](#validacionhtml)
      4. [`registro.html`](#registrohtml)
      5. [`edicion.html`](#edicionhtml)
      6. [`buscar.html`](#buscarhtml)
      7. [`historial.html`](#historialhtml)
      8. [`error.html`](#errorhtml)
   3. [`patients.db`](#patientsdb)
      1. [`records`](#records)
      2. [`history`](#history)
   4. [`requirements.txt`](#requirementstxt)
   5. [`app.py`](#apppy)
      1. [`index`](#index)
      2. [`history`](#history-function)
      3. [`search`](#search)
      4. [`edit`](#edit)
      5. [`editpatient`](#editpatient)
      6. [`register`](#register)
      7. [`validate`](#validate)
      8. [`errorhandling`](#errorhandling)
3. [Credits](#credits)
  
### Video DEMO

[My final project](https://youtube.com)
  
### Description

This web application is made completely in Python (see [Requirements](requirements.txt)), using a SQLite 3 database, involving HTML and CSS with the help of Jinja (Python's framework) to render the page's templates. The purpose of this web app is to keep track of every patient who comes to the Clinical Laboratory :microscope::test_tube::syringe: (specifically this one called "Violeta Zapata"), so the personal can register a patient whenever they come, and search them in the database when it's required; thus, every patient has their own records and it's easier to find them. The entire web app is displayed in Spanish to the user.

Next, every file implemented in this project is going to be explained:

#### `static`

The [static/](static/) folder contains four files:

##### `favicon.ico`

It's the icon to display on one side of the current tab (in the top of the browser).

##### `logo.png`

Which is going to be displayed in the index page.

##### `styles.css`

The project's style sheet, it contains custom classes and the styles for the project's tags. Among them are the styles for the `table`s, `nav` bar and `form-group`s.

##### `transparent.png`

This is the logo that is shown in the navigation bar.
  
#### `templates`

[This folder](templates/) contains all the HTML pages used in this project.

##### `layout.html`

Each one of these HTML files extends `layout.html` using Jinja, in other words, this is the project's template. It contains the `head` tag, the `nav` bar, a `header` which is going to be filled if the current page receives an alert, a `footer`, and the `main` block, which will contain the pages main info.  
It also checks the value of the `active` variable, depending on that, it will highlight the current page in the navigation bar.

##### `index.html`

The main page. It consists of the laboratory's logo and name. If the laboratory has some lab tests left to deliver, they're shown in this page; otherwise, it'll announce that there are no pending exams.

##### `validacion.html`

This page handles the patient's validation, it involves a form where the user can type the patient's ID in and submit it to verify if the user was already registered.

##### `registro.html`

After validation, we encounter the registration form. If the patient was already registered, the ID, first and last name, phone number, age and address fields are going to be filled automatically; otherwise the user will have to fill them here. Besides all those fields mentioned before, the user will have to select all the lab tests required and the total amount to pay (:money_with_wings:) before submitting.

##### `edicion.html`

Here you can change the patient's info. *Important*: changes will be applied to the [records](#records) table and will *not* affect the [`history`](#history) table's corresponding entries (except for the `user_id` field).

##### `buscar.html`

In this page you can search a patient by their ID (*cedula*) in order to look all their entries in the [`history`](#history) table and their corresponding info.

##### `historial.html`

This option lets the user see all the patients who have gone to the clinical lab so far. It shows only 20 patients per page.

##### `error.html`

Shows an error message in case some error pops up, and asks the user to go to the [main page](#indexhtml).

#### `patients.db`

This file represents a SQLite3 database that contains two tables:  

##### `records`

Contains all the user info, defined by the `id` (*cedula*) which is going to be unique for each user. This table was created by the query:

```sql
CREATE TABLE IF NOT EXISTS records 
(id INTEGER UNIQUE NOT NULL,
fname TEXT NOT NULL,
lname TEXT NOT NULL,
age INTEGER NOT NULL,
agetype TEXT NOT NULL,
address TEXT NOT NULL,
phone TEXT) 
```

Where:

- `id` is the patient's unique identifier (provided by the user in the register page (*registro*)),
- `fname` is the patient's first name,
- `lname` is the patient's last name,
- `age` is the patient's age,
- `agetype` is the patient's age type (it could be "years", "months" or "days"),
- `address` is the patient's address, and
- `phone` is the patient's phone number.

Every field is required, except for the phone number (*optional*).  

##### `history`

A table that contains every time an user goes to the laboratory; that means, a patient can appear **several times** in the `history` table, but they can only appear **once** in the `records` table. This `history` table was created by the query:

```sql
CREATE TABLE IF NOT EXISTS history
(entry_id INTEGER NOT NULL PRIMARY KEY,
user_id INTEGER NOT NULL,
exams TEXT NOT NULL,
year INTEGER NOT NULL,
month INTEGER NOT NULL,
day INTEGER NOT NULL,
age INTEGER NOT NULL,
agetype TEXT NOT NULL,
address TEXT NOT NULL,
phone TEXT,
total_amount INTEGER NOT NULL,
delivered TEXT,
FOREIGN KEY (user_id) REFERENCES records(id))
```

Where:

- `entry_id` is an unique row identifier for every entry,
- `user_id` is the `FOREIGN KEY` that joins the two tables (represents the `id` field from the `records` table),
- `exams` is a string that contains all the names of the tests the patient made in that specific entry,
- `year`, `month` and `day` are integers that represent the current date (year, month and day) when the patient went to the laboratory,
- `total_amount` represents the money that the patient has to pay, and
- `delivered` is a string that will contain the date and time when the patient withdraws the lab tests.

The fields `age`, `agetype`, `address` and `phone` are obtained from the `records` table.  
`delivered` and `phone` are the only fields that are not required in this table.  
The **difference** between these two tables is that the `history` table is **inmutable** (except when we `UPDATE` the `delivered` date), while the `records` table **can be modified** (see [`edicion.html`](#edicionhtml)).  
  
#### `requirements.txt`

This file contains all the modules needed for the application to run properly (assuming you have `python` and `pip` pre-installed in your computer).
  
#### `app.py`

[This file](app.py) is the project's controller, as the name suggests, it controls everything the website does in the back-end. Let's explain it in more detail:

The first thing it does is importing all the modules required (see [Requirements](requirements.txt)). Then, two lists are created, one called `nombres` and it stores all the lab tests names (displayed in the register or *registro* page), the other one's called `valores` and it contains the lab tests' abbreviations that are going to be saved in the database and displayed in the front-end tables.  

Next, the flask application is configured and some custom functions are created: `startdb` to start the database connection, it returns a connection object (`con`) and a cursor object (`cur`); the `closedb` function accepts these same two arguments in order to close the connection with the database.  

Besides, the function `get_patients` queries the database to get the patients that are going to be shown in the current page. You can see more about the `flask_paginate` module [here](https://gist.github.com/mozillazg/69fb40067ae6d80386e10e105e6803c9).

The `active` variable used in this file is an integer that represents the current page, a **1** represents the [register](#registrohtml) page, **2** represents the [search](#buscarhtml) page, **3** represents the [history](#historialhtml) page, and **4** represents the [edition](#editpatient) page. This way, the current page can be styled differently through the `active` class, defined in the [`styles.css`](#stylescss) file.

The following fuctions are going to be called every time the user accesses a given route:

##### `index`

The route `/` allows two methods. When the user visits the page through `GET`, the [`index.html`](#indexhtml) file is rendered, sending all the pending users (if any). If the user gets to the page through `POST`, it means that they already marked a lab test as delivered. Then the function checks validity before assigning the current date and time to the `delivered` field in the [`history`](#history) table.

##### `history` {#history-function}

After starting the connection with the database, the function `get_page_args` is called to get the current page's number, then the function `get_patients` is called, storing that value in the `patients` variable. Later on, the pagination object is created and all the collected info is returned with the `render_template` function.

##### `search`

When the user gets to the page through `GET`, the [`buscar.html`](#buscarhtml) file is rendered so the user can type the patient's ID in. Otherwise (if the request method was `POST`), checks if the submitted value is valid, and looks it up in the database, returning whatever was found.

##### `edit`

Similar to the [`validate`](#validate) function. Looks if the user was already registered in order to edit their info (verifying the user input first).

##### `editpatient`

Actually modifies the patient's info. It utilizes the same process as the [`register`](#register) function. If the `nuevacedula` (new ID) field is left blank or it's the same as the `cedula` (old ID) field, the patient's ID is not going to be modified in the database. If the ID is modified, it does so in both tables (`records` and `history`). See [`history`](#history) for more information.

##### `register`

This function handles the patient's registration. If the user reached the route via `GET`, it would redirect them to the `/validacion` route and let the [`validate`](#validate) handle it.  
After the user submits the registration form, this function will validate the data (ensuring every field is filled, and number values are positive). If this is the first time a patient comes to the clinical lab, it is registered into the `records` table and the `history` table, otherwise, it's inserted only into the `history` table.  
The phone numbers validation and formatting are handled by the `phonenumbers` module. You can learn more about it [here](https://stackabuse.com/validating-and-formatting-phone-numbers-in-python/).

##### `validate`

If the user reached this route via `GET`, the [`validacion.html`](#validacionhtml) template will be rendered. Else (user reached via `POST`), the function does the validity check, and looks the patient's ID up in the `records` table, then it passes all their info as an argument to the `render_template` function (rendering the [`registro.html`](#registrohtml) template), including the lab tests' names and values.

##### `errorhandling`

Whenever an `InternalServerError` or an `HTTPException` is raised, the function `flash`es the error and renders the [`error.html`](#errorhtml) template.

### Credits

Author: **Ana Verónica Aponte Martínez**  
Location: **Apure, Venezuela**
  
***This was CS50x!*** :woman_technologist::tada:
