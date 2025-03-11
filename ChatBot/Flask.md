``` python
books = [
    {
        'id': 1,
        'author': 'Bill Gates',
        'title': 'Creating of OS Windows',
        'year': 1998
    },
    {
        'id': 2,
        'author': 'Sergey Brinn',
        'title': 'Life in Google',
        'year': 2003
    },
    {
        'id': 3,
        'author': 'Elon Musk',
        'title': 'I wanna be on Mars',
        'year': 2017
    }
]

"""
▪ / - сайт
▪ /api/v1/ - API (да, я капитан)
▪ /api/v1/books/ - раздел сайта, отвечающий за книги. Получение списка
▪ /api/v1/books/<id>/ - получение определённой книги
"""

from flask import Flask, jsonify, abort

app = Flask(__name__)

@app.route('/', methods=['GET'])
def index_page():
    return "<h1>Hello, world</h1>" \
           "<p>This is Simple API"

@app.route('/api/v1/books/', methods=['GET'])
def books_list():
    return jsonify(books)

@app.route('/api/v1/books/<int:book_id>/', methods=['GET'])
def get_book(book_id):
    if book_id < 0 or book_id >= len(books):
        abort(404)
        # return jsonify({'status': 'Error', 'details': '404 Not Found'})
    return jsonify(books[book_id])

if __name__ == '__main__':
    app.run(debug=True)
```