---
layout: post
title: Build a RESTful Flask API - The TDD Way - scotch.io Tutorial 1 Notes
categories: code software development python flask
---

The <a href="https://scotch.io/tutorials/build-a-restful-api-with-flask-the-tdd-way">Build a RESTful Flask API - The TDD Way</a> tutorial by Jee Gikera is one of the best tutorials I have followed and seemed to meet an impossible combination of my learning requirements (Flask + API + TDD + Python + PostgreSQL + SQLAlchemy).  Jee's attention to detail and structuring an application to scale are two areas sometimes missing from other tutorials.

Below are my notes after completing the first tutorial.  The name of my project used throughout these notes is `strobla`.

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

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Gists via GitHub Pages

Vestibulum id ligula porta felis euismod semper. Nullam quis risus eget urna mollis ornare vel eu leo. Donec sed odio dui.

{% gist 5555251 gist.md %}

Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Nullam quis risus eget urna mollis ornare vel eu leo. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Donec sed odio dui. Vestibulum id ligula porta felis euismod semper.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](http://placehold.it/800x400 "Large example image")
![placeholder](http://placehold.it/400x200 "Medium example image")
![placeholder](http://placehold.it/200x200 "Small example image")

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/poole/poole/issues/new">Open an issue.</a>
