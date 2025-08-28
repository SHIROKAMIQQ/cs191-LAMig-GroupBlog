# How to Update Group Blog

## Environment Setup
- Clone this repository.
- Inside the directory:
```
pip install mkdocs
```

This blog uses [`mkdocs`](https://www.mkdocs.org/). In essence, this just turns Markdown into the body of a webpage.

## Adding a Page

Each `.md` file in the `docs` folder acts as a page of the blog.

## Running the blog

To host the blog locally, run:
```
$ mkdocs serve
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 0.41 seconds
INFO    -  [13:49:24] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO    -  [13:49:24] Serving on http://127.0.0.1:8000/mysite/
```

Copy the address on the last line, then paste it onto your browser.

To publish the update, push your updates into the github repository. 
