# Django Models and Migrations

## Learning Objectives - James

* Create a new Django application with Postgres as the default database
* Use `manage.py` commands to create, edit, update and seed a database
* Write models using Django and use them to modify the database tables.
* Look at Django's ORM.

## Framing

In this lesson, we will be focusing on the many features that Django provide us
to set up and maintain our database and models.

## We Do: Set Up a Django Application (10 minutes / 0:10) - Ali

Let's start by making a directory for our project:

```bash
$ mkdir django-starter && cd django-starter
```

Let's also build a virtual environment. Virtual environments allow us to have
multiple versions of Python on the same system so we can have different versions
of both Python and the packages we are using on our computers.

```bash
$ pip3 install virtualenv
$ virtualenv .env -p python3
$ source .env/bin/activate
```

Let's also install some dependencies and save them. Django doesn't utilize a
`Gemfile` or a `package.json`. Instead, we just use a text file that lists all
of our dependencies. The `pip freeze` command saves the dependencies in our `virtualenv` to
that file.

```bash
$ pip install Django==2.0.5
$ pip install psycopg2
$ pip freeze > requirements.txt
```
> You may need to also install `pip install psycopg2-binary`

Django is, of course, the framework we are using. `psycopg2` allows us to use
PostgreSQL within Django.

If you are downloading and running a Python project, you can usually install 
its dependencies with `pip install -r requirements.txt`.

Let's go ahead and create our project. `django-admin` gives us commands to
generate some of our project for us.

```bash
$ django-admin startproject tunr_django
$ cd tunr_django
```

Take a minute to look at the generated files.

Let's go ahead and also create our app.

```bash
$ django-admin startapp tunr
```
Again, take a minute to look at the newly generated files.

By default, Django uses MySQL for its database.  Let's use PostgreSQL instead
since it's better suited for production applications.

Create a database:

```bash
$ psql
> CREATE DATABASE tunr;
> CREATE USER tunruser WITH PASSWORD 'tunr';
> GRANT ALL PRIVILEGES ON DATABASE tunr TO tunruser;
> \q
```

Then, in `tunr_django/settings.py` find the `DATABASE` constant dictionary.
Let's edit it to look like this:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'tunr',
        'USER': 'tunruser',
        'PASSWORD': 'tunr',
        'HOST': 'localhost'
    }
}
```

We must also include the app we generated. On the bottom line of the
`INSTALLED_APPS` list, add `tunr`. Whenever you create a new app, you have to
include it in the project.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tunr'
]
```

Now, in the terminal run `python manage.py runserver` and then navigate to
`localhost:8000`. You should see a page welcoming you to Django!

`manage.py` contains a lot of management commands for Django. We'll see more
later, but [here](https://docs.djangoproject.com/en/1.11/ref/django-admin/) is
the full documentation if you are interested in what's going on behind the
scenes.

## Models - James

Let's start working with some data. In Django, we will write out models. 
Models represent the data layer of our application. We store that data in our
database:

[rMVC Diagram](http://i.stack.imgur.com/Sf2OQ.png)

First, lets create a Python class that inherits from the Django built in
`models.Model` class. Let's also define the fields within it. We do so like
this:

```python
# tunr/models.py
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()
```

Here, we are defining three fields (which will be represented as columns in our
database): `name`, `photo_url` and `nationality`. `name` and `nationality` are
character fields which means that we must add an upper limit to how many
characters are in that database field. The `photo_url` will have unlimited
length. The full listing of the available fields are
[here](https://docs.djangoproject.com/en/1.11/ref/models/fields/).

Let's also add the magic method `__str__`. This method defines what an instance
of the model will show up as by default. It will be really helpful for debugging
in the future!

```python
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()

    def __str__(self):
        return self.name
```

## Migrations - Ali

In the SQL class, we talked about how schema is enforced on the database side when we use SQL databases. But here we are writing our schema on the Python side! We have to translate that code into the schema for our database. We will do so using migrations. In some frameworks, you have to write your migrations yourself, but in Django the framework writes them for us! 

In order to migrate this model to the database, we will run two commands. The first is:

```bash
$ python manage.py makemigrations
```

This will generate a migration file that gets all of its data from the code in
the `models.py` file. You normally don't edit these files manually in Django.

Let's also run:

```bash
$ python manage.py migrate
```

This will commit the migration to the database.

### Foreign Keys - James

Let's also start filling out the Song model. We will define the class and then
add a foreign key. We do so like this:

```python
class Song(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE, related_name='songs')
```

The `related_name` refers to how the model will be referred to in relation to
its parent -- you will see this in use later on. `on_delete` specifies how we
want the models to act when their parent is deleted. By using cascade, related
children will be deleted.

### Admin Console - Ali

Before we get too far, let's also create a superuser for our app. Django has
authentication (and authorization) right out of the box, so you don't have to
write it yourself or add a plugin.

In the terminal, run:

```bash
$ python manage.py createsuperuser
```

Then fill in the information in the boxes that pop up!

So far in this class, we have used seed files to add initial data to our
databases. We can also do that in Django ([see this
article](https://docs.djangoproject.com/en/1.11/howto/initial-data/)), but let's
try something a little bit different instead.

Django has an admin dashboard built in, which gives us full CRUD functionality
straight out of the box.

Let's set it up! In `tunr/admin.py`, add the following code:

```python
from django.contrib import admin
from .models import Artist

admin.site.register(Artist)
```

If you now navigate to `localhost:8000/admin`, you get a full admin view where
you have full CRUD functionality for your model! Create two Artists here.

### You Do: Add the Song model (5 minutes) - James

Add `title`, `album` and `preview_url` fields, then create and run the
migrations. Finally create three songs using the admin site.


## Django's ORM - Ali

Django has an ORM, similar to Mongoose in Express. Let's look at a few queries.

```python
# Select all of the artist objects in the database
Artist.objects.all()

# Get the artist with the id of 1 (can also do pk here which stands for primary key)
Artist.objects.get(id=1)

# Get the artist with the name "Kanye West", if there are two Kanye's this will error!
Artist.objects.get(name="Kanye West")

# Get all the Artists who are from the USA
Artist.objects.filter(nationality="USA")

# Create an Artist with the following attributes, then save, commiting it to the database
kanye = Artist(name="Kane West", photo_url="test.com", nationality="USA")
kanye.save()

# Oops, we misspelled Kanye's name! Let's change it and then commit to the DB
kanye.name = "Kanye West"
a.save()

# Let's add a song to the artist
song = Song(title="Ultralight Beam", album="The Life of Pablo", preview_url="test.com", artist=a)
song.save()

# Delete the song
song.delete()
```

These are some simple ones, but if you like Django, know that you can do some
really cool things with it's ORM. For example:

```python
# This will return all Artists who's name starts with an A
Artist.objects.filter(name__startswith="A")

# This will return all the songs with the id's 1 and 2, excluding all those equal to or greater than 3
Song.objects.exclude(artist_id__gte=3)
```

If you want to access a REPL, run `$ python manage.py shell`. 

## Bonus: Django Extensions

Django Extensions adds additional debugging functionality to Django. We would **highly** recommend using it to make coding easier! [Link](https://github.com/django-extensions/django-extensions).

To set it up:

```
$ pip install django-extensions
```

Add `django_extensions` to your `INSTALLED_APPS` list:

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tunr',
    'django_extensions'
]
```

You can now run `python manage.py shell_plus --ipython` to get to an IPython shell.

## Closing/Questions (10 minutes / 2:30)

## Homework

Complete the Models + Migrations portion of
[Scribble](https://git.generalassemb.ly/dc-wdi-python-django/scribble).

## Additional Resources

* [Django Docs: Models](https://docs.djangoproject.com/en/2.0/topics/db/models/)
* [Django Docs: Models & Databases](https://docs.djangoproject.com/en/2.0/topics/db/)
* [How to Create Django Models](https://www.digitalocean.com/community/tutorials/how-to-create-django-models)
* [Django Docs: Migrations](https://docs.djangoproject.com/en/2.0/topics/migrations/)
* [Django Docs: Writing Database Migrations](https://docs.djangoproject.com/en/2.0/howto/writing-migrations/)
* [Django Docs: Providing initial data for models](https://docs.djangoproject.com/en/1.11/howto/initial-data/)
* [Django Extensions](https://github.com/django-extensions/django-extensions)

## Sample Quiz Questions
