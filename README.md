Dominate
========

`Dominate` is a Python library for creating and manipulating HTML documents using an elegant DOM API.
It allows you to write HTML pages in pure Python very concisely, which eliminate the need to learn another template language, and to take advantage of the more powerful features of Python.


Compatability
-------------

`Dominate` is compatable with both Python 2.7 and Python 3.3. There are known issues with Python 3.2 and below.

Installation
------------

The recommended way to install `dominate` is with
[`pip`](http://pypi.python.org/pypi/pip/):

    sudo pip install dominate


Developed By
------------

* Tom Flanagan - <tom@zkpq.ca>
* Jake Wharton - <jakewharton@gmail.com>

Git repository located at
[github.com/Knio/dominate](http://github.com/Knio/dominate)


Examples
========

All examples assume you have imported the appropriate tags or entire tag set (i.e. `from dominate.tags import *`).


Hello, World!
-------------

The most basic feature of `dominate` exposes a class for each HTML element, where the constructor
accepts child elements, text, or keyword attributes. `dominate` nodes return their HTML representation
from the `__str__`, `__unicode__`, and `render()` methods.

    >>> print html(body(h1('Hello, World!')))
    <html>
      <body>
        <h1>Hello, pyy!</h1>
      </body>
    </html>


Attributes
----------

`Dominate` can also use keyword arguments to append attributes onto your tags. Most of the attributes are a direct copy from the HTML spec with a few variations.

Use `cls` for class names. `cls` is used because `class` in python is a [reserved keyword](http://docs.python.org/2/reference/lexical_analysis.html#keywords "Reserved Keywords").

    >>> test = div(cls='classname anothername')
    >>> print test
    <div class="classname anothername">
    </div>

Use `data_*` for [custom HTML5 data attributes](http://www.w3.org/html/wg/drafts/html/master/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes "HTML5 Data Attributes").

    >>> test = div(data_employee='101011')
    >>> print test
    <div data-employee="101011">
    </div>

You can also modify the attributes of tags through a dictionary-like interface:

    >>> header = div()
    >>> header['id'] = 'header'
    >>> print header
    <div id="header"></div>


Complex Structures
------------------

Through the use of the `+=` operator and the `.add()` method you can easily create more advanced structures.

Create a simple list:

    >>> list = ul()
    >>> for item in range(4):
    ...   list += li('Item #', item)
    ...
    >>> print list
    <ul>
      <li>Item #0</li>
      <li>Item #1</li>
      <li>Item #2</li>
      <li>Item #3</li>
    </ul>

If you are using a database or other backend to fetch data, `dominate` supports iterables to help streamline your code:

    >>> print ul(li(a(name, href=link), __inline=True) for name, link in menu_items)
    <ul>
      <li><a href="/home/">Home</a></li>
      <li><a href="/about/">About</a></li>
      <li><a href="/downloads/">Downloads</a></li>
      <li><a href="/links/">Links</a></li>
    </ul>

A simple document tree:

    >>> _html = html()
    >>> _body = _html.add(body())
    >>> header  = _body.add(div(id='header'))
    >>> content = _body.add(div(id='content'))
    >>> footer  = _body.add(div(id='footer'))
    >>> print _html
    <html>
      <body>
        <div id="header"></div>
        <div id="content"></div>
        <div id="footer"></div>
      </body>
    </html>

For clean code, the `.add()` method returns children in tuples. The above example can be cleaned up and expanded like this:

    >>> _html = html()
    >>> _head, _body = _html.add(head(title('Simple Document Tree')), body())
    >>> names = ['header', 'content', 'footer']
    >>> header, content, footer = _body.add(div(id=name) for name in names)
    >>> print _html
    <html>
      <head>
        <title>Simple Document Tree</title>
      </head>
      <body>
        <div id="header"></div>
        <div id="content"></div>
        <div id="footer"></div>
      </body>
    </html>

You can modify the attributes of tags through a dictionary-like interface:

    >>> header = div()
    >>> header['id'] = 'header'
    >>> print header
    <div id="header"></div>

Or the children of a tag though an array-line interface:

    >>> header = div('Test')
    >>> header[0] = 'Hello World'
    >>> print header
    <div>Hello World</div>


Comments can be created using objects too!

    >>> print comment('BEGIN HEADER')
    <!--BEGIN HEADER-->
    >>> print comment(p('Upgrade to newer IE!'), condition='lt IE9')
    <!--[if lt IE9]>
    <p>Upgrade to newer IE!</p>
    <![endif]-->


Context Managers
----------------

You can also add child elements using Python's `with` statement:

    >>> h = ul()
    >>> with h:
    ...   li('One')
    ...   li('Two')
    ...   li('Three')
    ...
    >>>
    >>> print h
    <ul>
      <li>One</li>
      <li>Two</li>
      <li>Three</li>
    </ul>


You can use this along with the other mechanisms of adding children elements, including nesting `with` statements, and it works as expected:

    >>> h = html()
    >>> with h.add(body()).add(div(id='content')):
    ...   h1('Hello World!')
    ...   p('Lorem ipsum ...')
    ...   with table().add(tbody()):
    ...     l = tr()
    ...     l += td('One')
    ...     l.add(td('Two'))
    ...     with l:
    ...       td('Three')
    ...
    >>>
    >>> print h
    <html>
      <body>
        <div id="content">
          <h1>Hello World!</h1>
          <p>Lorem ipsum ...</p>
          <table>
            <tbody>
              <tr>
                <td>One</td>
                <td>Two</td>
                <td>Three</td>
              </tr>
            </tbody>
          </table>
        </div>
      </body>
    </html>


When the context is closed, any nodes that were not already added to something get added to the current context.

Attributes can be added to the current context with the `attr` function:

    >>> d = div()
    >>> with d:
    ...   attr(id='header')
    ...
    >>>
    >>> print d
    <div id="header"></div>


Decorators
----------

`Dominate` is great for creating reusable widgets for parts of your page. Consider this example:

    >>> def greeting(name):
    ...     with div() as d
    ...         p('Hello, %s' % name)
    ...     return d
    ...
    ...
    >>> print greeting('Bob')
    <div>
      <p>Hello, Bob</p>
    </div>
    >>>

You can see the following pattern being repeated here:

    def widget(parameters):
      with tag() as t:
          ...
      return t

This boilerplate can be avoided by using tags (objects and instances) as decorators

    >>> @div
    ... def greeting(name):
    ...     p('Hello %s' % name)
    ...
    ...
    >>> print greeting('Bob')
    <div>
      <p>Hello Bob</p>
    </div>
    >>>

The decorated function will return a new instance of the tag used to decorate it, and execute in a `with` context which will collect all the nodes created inside it.

You can also use instances of tags as decorators, if you need to add attributes or other data to the root node of the widget.
Each call to the decorated function will return a copy of the node used to decorate it.

    >>> @div(h2('Welcome'), cls='greeting')
    ... def greeting(name):
    ...     p('Hello %s' % name)
    ...
    ...
    >>> print greeting('Bob')
    <div class="greeting">
      <h2>Welcome</h2>
      <p>Hello Bob</p>
    </div>
    >>>



Creating Documents
------------------

Since creating the common structure of an HTML document everytime would be excessively tedious dominate provides a class to create an manage them for you, `document`.

When you create a new document, the basic HTML tag structure is created for you.

    >>> d = document()
    >>> print d
    <!DOCTYPE html>
    <html>
      <head>
        <title>PYY Page</title>
      </head>
      <body></body>
    </html>

The `document` class accepts `title`, `doctype`, and `request` keyword arguments.
The default values for these arguments are `Dominate`, `<!DOCTYPE html>`, and `None` respectively.

The `document` class also provides helpers to allow you to access the `html`, `head`, and `body` nodes directly.

    >>> d = document()
    >>> d.html
    <dominate.tags.html: 0 attributes, 2 children>
    >>> d.head
    <dominate.tags.head: 0 attributes, 0 children>
    >>> d.body
    <dominate.tags.body: 0 attributes, 0 children>

You should notice that here the `head` tag contains zero children.
This is because the default `title` tag is only added when the document is rendered and the `head` element already does not explicitly contain one.

The `document` class also provides helpers to allow you to directly add nodes to the `body` tag.

    >>> d = document()
    >>> d += h1('Hello, World!')
    >>> p += p('This is a paragraph.')
    >>> print d
    <!DOCTYPE html>
    <html>
      <head>
        <title>PYY Page</title>
      </head>
      <body>
        <h1>Hello, World!</h1>
        <p>This is a paragraph.</p>
      </body>
    </html>

