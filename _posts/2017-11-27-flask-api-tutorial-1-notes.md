---
layout: post
title: Build a RESTful Flask API - The TDD Way - scotch.io Tutorial 1 Notes
categories: code software development python flask
---

The <a href="https://scotch.io/tutorials/build-a-restful-api-with-flask-the-tdd-way">Build a RESTful Flask API - The TDD Way</a> tutorial by Jee Gikera is one of the best tutorials I have followed and seemed to meet an impossible combination of my learning requirements (Flask + API + TDD + Python + PostgreSQL + SQLAlchemy).  Jee's attention to detail and structuring an application to scale are two areas sometimes missing from other tutorials.

Below are my notes after completing the first tutorial.  The name of my project used throughout these notes is `strobla` and `Activity` is the data model.

> Where's my project GitHub link?  I am using <a href="https://bitbucket.org">Atlassian Bitbucket</a> as a free private source code repository whilst my project is in its early stages.  Open sourcing my project, either through Bitbucket or GitHub, is a decision I'll make later..

# Virtual Environment

On Windows, `autoenv` did not set environment variables from `.env` when I `cd` into the project folder.

After trying alternatives (`source`, etc), I created the following file in my home directory:

{% highlight powershell %}
# C:\Users\naaman\.env_strobla.ps1

$env:PROJECT_HOME="C:\Users\naaman\OneDrive\Documents\Code\strobla-python"
cd $env:PROJECT_HOME
.\venv\Scripts\activate.ps1
$env:FLASK_APP="run.py"
$env:SECRET="secret"
$env:ROLLBAR_TOKEN="token"
$env:APP_STAGE="dev"
$env:DATABASE_URL="postgresql://strobla:password@localhost/strobla"
$env:DATABASE_URL_TESTING="postgresql://strobla:password@localhost/strobla_testing"
atom
{% endhighlight %}

The .env_<project>.ps1 file can be easily run each time you open a new PowerShell console.  The file also helps to keep secrets out of the version-controlled project directory.

> `APP_STAGE` replaces `APP_SETTINGS` in my project and was changed as a personal preference and readability.  Throughout my notes, I will make minor changes to variable names however I recommend using the same directory structures and naming conventions.

# Environment Configurations

I pointed the Testing database URI to a new environment variable to keep passwords out of source control.  Also note the different environment names I am using..

{% highlight python %}
# instance\config.py

import os

class Config(object):
    """ Parent configuration class """
    DEBUG = False
    CSRF_ENABLED = True
    SECRET = os.getenv('SECRET')
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL')

class TestingConfig(Config):
    """ Testing environment configuration """
    """  - with a separate test database """
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL_TESTING')
    DEBUG = True

class DevConfig(Config):
    """ Dev stage configuration """
    DEBUG = True

class QAConfig(Config):
    """ QA stage configuration """
    DEBUG = True

class ProdConfig(Config):
    """ Prod stage configuration """
    DEBUG = False
    TESTING = False

app_config = {
    'testing': TestingConfig,
    'dev': DevConfig,
    'qa': QAConfig,
    'prod': ProdConfig,
}
{% endhighlight %}

# Data Model

My project uses retrieves JSON data from the <a href="https://developers.strava.com/docs/reference/">Strava API</a> and saves it into the Postgres database.  For simplicity, my table contains two columns:

* `id` : Derived from the JSON data instead of auto creating new IDs
* `data` : a JSONB type Postgres column to store the JSON object

When issuing PUT API calls, I was getting <em>Duplicate Key</em> errors.  I added an `update()` function to the model to support all `PUT` calls.  An alternative is to use `Activity.query.filter_by(id=id).update()` within `app\__init__.py`.

{% highlight python %}
# app\models.py

from app import db
from sqlalchemy.dialects.postgresql import JSONB
import json

class Activity(db.Model):
    """ This class represents the Strava activities table """

    __tablename__ = 'activities'

    id = db.Column(db.Integer, primary_key=True)
    data = db.Column(JSONB)
    date_created = db.Column(db.DateTime, default=db.func.current_timestamp())
    date_modified = db.Column(
        db.DateTime,
        default=db.func.current_timestamp(),
        onupdate=db.func.current_timestamp())

    def __init__(self, data):
        """ initialise with activity id & JSON data """
        self.id = data['id']
        self.data = data

    def save(self):
        db.session.add(self)
        db.session.commit()

    def update(self):
        id_from_data = self.data['id']
        stmt = self.__table__.update().\
            where(self.id==id_from_data).\
            values(data=self.data)
        result = db.session.execute(stmt)
        db.session.commit()
        return result

    ### AS PER TUTORIAL ###

{% endhighlight %}

# API Functionality

A note separate to the tutorial is the handling of JSON data through the API.  When submitting JSON through the API, eg. `self.client().put('/activities/id, data={JSON}')`, the JSON must be wrapped in a `json.dumps()` call to avoid Flask ValueErrors.  Similarly, use `json.loads()` to convert back to the original JSON format.

A minor change was made to the `PUT` section to use `activity.update()` instead of `.save()` and the `DELETE` section was changed as follows as I was unable to get the tutorial code to work:

{% highlight python %}
# app\__init__.py

        if request.method == 'DELETE':
            activity.delete()
            response = jsonify({
                'message': 'activity {} deleted'.format(activity.id)
            })
            response.status_code = 200
            return response

{% endhighlight %}

# Conclusion

The tutorial helps understand how the numerous pieces fit together in a fully-featured Flask-based API application.  I was able to adapt my project easily to fit within this architecture and continue learning how to write Python code.  Looking forward to <a href="https://scotch.io/tutorials/build-a-restful-api-with-flask-the-tdd-way-part-2">Part 2</a>..
