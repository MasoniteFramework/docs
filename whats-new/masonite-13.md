# Masonite 1.3

# Introduction

Masonite 1.3 comes with a plethora of improvements over previous versioning. This version brings new features such as Queue and Mail drivers as well as various bug fixes.

## 1. Fixed infinite redirection

Previously when a you tried to redirect using the `Request.redirect()` method, Masonite would sometimes send the browser to an infinite redirection. This was because masonite was not resetting the redirection attributes of the `Request` class.

## 2. Fixed browser content length 

Previously the content length in the request header was not being set correctly which led to the gunicorn server showing a warning that the content length did not match the content of the output.

## 3. Changed how request input data is retrieved

Previously the Request class simply got the input data on both `POST` and `GET` requests by converting the `wsgi.input` WSGI parameter into a string and parsing. All POST input data is now retrieved using `FieldStorage` which adds support for also getting files from `multipart/formdata` requests.

## 4. Added Upload Drivers

You may now simply upload images to both disk and Amazon S3 storage right out of the box. With the new `UploadProvider` service provider you can simply do something like:

```html
<html>
  <body>
      <form action="/upload" method="POST" enctype="multipart/formdata">
        <input type="file" name="file">
      </form>
  </body>
</html>
```

```python
def show(self, Upload):
    Upload.store(Request.input('file'))
```

As well as support for Amazon S3