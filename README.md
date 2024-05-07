# PyGenUz: Python Web Framewrok build for learning purposes

![purpose](https://img.shields.io/badge/purpose-learning-green)
![PyPI Version](https://img.shields.io/pypi/v/pygenuz)

PyGenUz is a Python web framewrok built for learning purposes.

It's a WSGI framework and can be used with any WSGI aplication server such as Gunicorn.

## Installation
```shell
pip install pygenuz
```
## How to use it

### Basic usage
    ```
    from pygenuz.app import PyGenUz

    app = PyGenUz()


    @app.route('/home', allowed_methods="get")
    def home(request, response):
        response.text = "Hello from home page"


    @app.route('/about')
    def about(request, response):
        response.text = "Hello from about page"


    @app.route("/hello/{name}")
    def greeting(request, response, name):
        response.text = f"Hello, {name}"


    @app.route('/books')
    class Books:
        def get(self, request, response):
            response.text = "Books page"

        def post(self, request, response):
            response.text = "endpoint to create a book"

        def delete(self, request, response):
            response.text = "deleted"
    ```

## How we can use Templates? 
### Using Templates
#### We have to make dir name called temps/ 
```
    @app.route("/temp")
    def template_handler(req, resp):
        resp.html = app.template(
            "home.html",
            context = {"new_title": "New title", "new_body": "New body"}
        )

```

## How we can use json? 

```
    @app.route("/json")
    def json_handler(req, resp):
        response_data = {"name": "some name"}
        resp.json = response_data

``` 

## How we can use Static files 
### We have to make dir name called static/

#### .html
```
    <link rel="stylesheet" href="/static/test.css">
```

## MODELS AND ORM
#### For create models you have to make only models.py and you must to write the following code.
```
from pygenuz.db import *

Base = declarative_base() 
```

##### Example models.py 
```
from pygenuz.db import *

Base = declarative_base() 

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    username = Column(String(50), nullable=False)
    email = Column(String(100), nullable=False, unique=True)
    age = Column(Integer)
    is_active = Column(Boolean, default=True)

class Book(Base):
    __tablename__ = 'books'

    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    author = Column(String(100), nullable=False)
    published_date = Column(DateTime)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship('User', back_populates='books')
```

#### After writing models.py, you must to write configure.py to set the database settings
```
from pygenuz.configs import *
from models import Base

def configure_database():
    if Base:
        DATABASE_URL = 'sqlite:///database.db'
        engine = create_engine(DATABASE_URL)
        Session = sessionmaker(bind=engine)
        session = Session()

        Base.metadata.create_all(engine)
        return session 
    else:
        return None

if __name__ == "__main__":
    configure_database()
```
#### After writing configure.py, you need to migrate your models to the database. To do this, you need to write a command in the terminal or cmd
```
python configure.py 
```
##### will return you a Migrated response if successful else error

## Using models.py in your views.py or main.py, you must import these codes.
```
from pygenuz.app import PyGenUz
from configure import *
app = PyGenUz()
session = configure_database()
```

#####  List in jinja
```
@app.route("/books", allowed_methods=["get"])
def list_books(request, response):
    books = session.query(Book).all()
    response.html = app.template(
        "books.html",
        context={"books": books}
    )
```
###### books.html file
```
    <h1>Book List</h1>
    <table>
        <thead>
            <tr>
                <th>Title</th>
                <th>Author</th>
                <th>Published Date</th>
            </tr>
        </thead>
        <tbody>
            {% for book in books %}
            <tr>
                <td>{{ book.title }}</td>
                <td>{{ book.author }}</td>
                <td>{{ book.published_date }}</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
```
##### Get json data 
```
@app.route("/api/books", allowed_methods=["get"])
def get_books(request, response):
    session = configure_database()
    books = session.query(Book).all()
    books_data = [
        {
            "id": book.id,
            "title": book.title,
            "author": book.author,
            "published_date": str(book.published_date)
        }
        for book in books
    ]
    response.json = books_data
```
##### POST json data 
```
@app.route("/api/book", allowed_methods=["post"])
def create_book(request, response):
    session = configure_database()
    book_data = request.json
    new_book = Book(
        title=book_data.get('title'),
        author=book_data.get('author'),
        published_date=book_data.get('published_date')
    )

    session.add(new_book)
    session.commit()

    response.text = "Book created successfully"
```
##### UPDATE and DELETE json data 
```
@app.route("/api/books/{book_id}", allowed_methods=["put", "delete"])
def book_api(request, response, book_id):
    if request.method == "PUT":
        update_book(request, response, book_id)
    elif request.method == "DELETE":
        delete_book(request, response, book_id)

def update_book(request, response, book_id):
    book_data = request.json
    book = session.query(Book).filter_by(id=book_id).first()
    if not book:
        response.text = f"Book with ID {book_id} not found"
        response.status_code = 404
        return
    book.title = book_data.get('title')
    book.author = book_data.get('author')
    book.published_date = book_data.get('published_date')
    session.commit()
    response.text = "Book updated successfully"

def delete_book(request, response, book_id):
    book = session.query(Book).filter_by(id=book_id).first()
    if not book:
        response.text = f"Book with ID {book_id} not found"
        response.status_code = 404
        return
    session.delete(book)
    session.commit()
    response.text = "Book deleted successfully"
```

# EVERY MONDAY YOU WILL SEE A NEW BIG UPDATE UNTIL V1.0.0 