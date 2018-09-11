## Deploying a Flask app using AWS

In this blog we will create a basic Flask application and deploy this to the web using AWS Lambda.  The application is a simple API that takes as input some user defined data and posts the output.  The inputs will also be saved to a database using AWS RDS.

Our simple API will calculate the roots of a given quadratic equation.  It will take as input three numerical values which represent the coefficients and calculate the values (if they exist).  We will also add in a way to store a record of these values in a database.

The full code for this blog can be found on [GitHub](https://github.com/imrankhan17/flask-serverless).

### Getting started

To begin with I'm going to create a new directory for this project:

```bash
mkdir flask-app-tutorial
cd flask-app-tutorial
```

Create a virtual environment and install the packages we will need.

```bash
python3 -m venv env
source env/bin/activate
pip install Flask Flask-WTF PyMySQL
```

We will need to create a function that calculates the roots of a quadratic equation.  We need to make sure that the function handles inputs of string type and that it outputs a string.

```python
def calculate_roots(a, b, c):
    """
    Calculates roots of quadratic equation of form `ax^2 +bx + c = 0`
    :param a: coefficient of x^2
    :param b: coefficient of x
    :param c: constant
    :return: roots if they exist
    """
    a, b, c = map(float, (a, b, c))

    if a == 0:
        return str(-c / b)

    discriminant = b**2 - 4 * a * c
    if discriminant < 0:
        return 'No real roots'

    x1 = (-b + discriminant ** 0.5) / (2 * a)
    x2 = (-b - discriminant ** 0.5) / (2 * a)

    if x1 == x2:
        return str(x1)

    return '{},{}'.format(x1, x2)
```

Let's put the above code in a file called `utils.py`.

We will also need another file called `app.py` to set up our Flask application.

```python
from flask import Flask, request
from utils import calculate_roots

app = Flask(__name__)

@app.route('/')
def result():
    return calculate_roots(a=request.args['a'], b=request.args['b'], c=request.args['c'])

if __name__ == '__main__':
    app.run()
```

`@app.route('/')` means our web-page is stored at the root URL i.e. the homepage.  `request` is a global Flask [variable](http://flask.pocoo.org/docs/1.0/reqcontext/) that keeps track of various data during a session.  In our case it's a, b and c that are stored in a dictionary called `args`.

We can test our app locally by simply running `python app.py`.  In our browser we can then go to `http://127.0.0.1:5000/` or localhost port 5000.

However, we get a '400 Bad Request' error.  That's because we haven't given it any input data.  To do this we have to pass in some URL arguments.  For example, try going to `http://127.0.0.1:5000/?a=1&b=-3&c=2`.  Now we see the expected output.

### Creating a user interface

So we have a minimum viable product working.  However, it's nor particularly user-friendly to input URL parameters to interact with the API.  Let's add a basic form for the user to fill in.  Create a new file called `form.py`.

```python
from flask_wtf import FlaskForm
from wtforms import DecimalField

class CoefficientsForm(FlaskForm):
    a = DecimalField('a')
    b = DecimalField('b')
    c = DecimalField('c')
```

We also need a HTML file to define what the form looks like and add a submit button.  Create a new folder called `templates` and add the file `form.html`.

```html
<form action="{{ url_for('calculate') }}" method="post">
    <a>a=</a>
    <input type="text" name="a">
    <a>b=</a>
    <input type="text" name="b">
    <a>c=</a>
    <input type="text" name="c">
    {{ form.csrf_token }}
    <input type="submit">
</form>
```

In `app.py` we need to add another function that will hold the form and forward the user-inputted data.  We import three more Flask functions: `render_template` which uses the HTML from above to create the page, `url_for` which generates the full URL for a particular page, and `redirect` which takes the user to a different page when the form is submitted.  The new code is as below:

```python
from flask import Flask, request, render_template, redirect, url_for
from form import CoefficientsForm
from utils import calculate_roots

app = Flask(__name__)
app.config['SECRET_KEY'] = '123'

@app.route('/', methods=['GET', 'POST'])
def calculate():
    form = CoefficientsForm()
    if form.validate_on_submit():
        return redirect(url_for('result', a=form.data['a'], b=form.data['b'], c=form.data['c']))
    return render_template('form.html', form=form)

@app.route('/result')
def result():
    return calculate_roots(a=request.args['a'], b=request.args['b'], c=request.args['c'])

if __name__ == '__main__':
    app.run()
```

We need to add a secret key to the app config to protect against [CSRF](https://flask-wtf.readthedocs.io/en/stable/csrf.html) attacks.  Within the `calculate` function we instantiate our form class, then we check if the form inputs are valid.  If they are, we redirect the user to the URL we constructed manually last time.  If it's not valid, we simply render the empty form page.  Note, we've also renamed the URL route of the `result` function to `/result`.

You can now try it out again with `python app.py`.

### Moving our app to the internet

So we have our basic application working with some user-friendly features.  Let's now share it with the rest of the world by deploying it to the world.  To do this, we use the Python package [Zappa](https://github.com/Miserlou/Zappa) that makes the whole process so much easier.

First, install the package with `pip install zappa`.  Also make sure you've got an AWS account and the [CLI tool](https://github.com/Miserlou/Zappa#installation-and-configuration) set-up locally.  Once all of that is done, enter `zappa init` and click through accepting all the defaults.  This will create a `zappa_settings.json` file in your repository.  Finally, enter `zappa deploy` and that's it - your application is now live on the web.  Click on the link provided and test it out.

### Saving input data to a database

Finally, as an extra feature let's incorporate a database into our app to store the values that are entered every time.

You can use the AWS RDS console to create a database, or the CLI command as below:

```bash
aws rds create-db-instance \
    --engine mariadb \
    --db-instance-class db.t2.micro \
    --allocated-storage 20 \
    --db-instance-identifier flask-blog \
    --master-username test_user \
    --master-user-password dont_use_this_password \
    --db-name mydb \
    --publicly-accessible
```

It will take about 5 minutes for the database to be created.  Once that is done, connect to the database and create a table to store the values for a, b, c and a timestamp.

```sql
USE mydb;

CREATE TABLE inputs (
  id int(11) NOT NULL AUTO_INCREMENT,
  a double DEFAULT NULL,
  b double DEFAULT NULL,
  c double DEFAULT NULL,
  timestamp timestamp(3) NOT NULL DEFAULT current_timestamp(3),
  PRIMARY KEY (id),
  UNIQUE KEY inputs_id_uindex (id)
);
```

We will now need to write a function to insert the data into our table every time someone uses our application.  Add the following to our `utils.py` file:

```python
import pymysql.cursors

def save_to_db(request):
    db = pymysql.connect(host='flask-blog.coprcosrifdf.eu-west-2.rds.amazonaws.com', user='test_user',
                         passwd='dont_use_this_password', port=3306)
    cur = db.cursor()
    cur.execute('INSERT INTO mydb.inputs (a, b, c) VALUES (%s, %s, %s);',
                (request.args['a'], request.args['b'], request.args['c']))
    db.commit()
    db.close()
```

Note, your `host` value will be different.  Also, ideally we would store our database credentials as [environment variables](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html) so they're not exposed in our code.

Back in `app.py`, we just need to add `save_to_db(request=request)` just before the return line in the `result` function.  Remember to import `save_to_db` from `utils`.

Now let's try it out with `python app.py`.  Put in some values then `select * from mydb.inputs;` to see if it worked.

We've confirmed it works locally so we just need to deploy it again to the web.  Zappa makes this easy with `zappa update`.  Now just go the site as before and try it out from there.

---
[Home](../index.md)
