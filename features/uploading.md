Masonite comes with a simple way to upload files to different file systems. 

In Masonite, a Filesystem is a place where assets are stored. These locations could be somewhere local on the server or in a cloud storage service like Amazon S3.

# Configuration

The configuration for the filesystem feature is broken up into "disks". These "disks" or connection configurations can be any name you want.

Here is an example configuration:

```python
DISKS = {
    "default": "local",
    "local": {
        "driver": "file",
        "path": os.path.join(os.getcwd(), "storage/framework/filesystem")
        #
    },
    "s3": {
        "driver": "s3",
        "client": os.getenv("AWS_CLIENT"),
        "secret": os.getenv("AWS_SECRET"),
        "bucket": os.getenv("AWS_BUCKET"),
    },
}
```

## Default

The default configuration is the name of the disk you want to be used as the default when using the Filesystem features.

## Local Driver

The local driver is used for local filesystems like server directories. All files are stored and managed "locally" on the server.

| Option   | Description                        |
| -------- | ---------------------------------- |
| `driver` | The driver to use for this disk    |
| `path`   | The base path to use for this disk |

## S3 Driver

The S3 driver is used for connecting to Amazon's S3 cloud service.

| Option   | Description                     |
| -------- | ------------------------------- |
| `driver` | The driver to use for this disk |
| `client` | The Amazon S3 client key        |
| `secret` | The Amazon S3 secret key        |
| `bucket` | The Amazon S3 bucket name       |

# Uploading Files

Uploading files is simple. You will have to use the Masonite `Storage` class. 

The first and most simplest method is taking a file and putting text into it. For this we can use the `put` method

```python
from masonite.filesystem import Storage
from masonite.requests import Request

def store(self, storage: Storage, request: Request):
  storage.disk('local').put('errors/info.log', 'errors')
```

| Method                                    | Description                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| `exists(file_path)`                       | Boolean to check if a file exists.                           |
| `missing(file_path)`                      | Boolean to check if a file does not exist.                   |
| `stream`                                  | Creates a `FileStream` object to manage a file stream.       |
| `copy('/file1.jpg', '/file2,jpg')`        | Copies a file from 1 directory to another directory          |
| `move('/file1.jpg', '/file2,jpg')`        | Moves a file from 1 directory to another                     |
| `prepend('file.log', 'content')`          | Prepends content to a file                                   |
| `append('file.log', 'content')`           | Appends content to a file                                    |
| `put('file.log', 'content')`              | Puts content to a file                                       |
| `put_file('directory', resource, "name")` | Puts a file resource to a directory. Must be an instance of Masonite's `UploadedFile`. Takes an optional third `name` parameter to specify the file name |

## Uploading Form Files

When uploading files from a form you will find the `put_file` method more useful:

```python
from masonite.filesystem import Storage
from masonite.requests import Request

def store(self, storage: Storage, request: Request):
  path = storage.disk('local').put_file('avatars', request.input('avatar'))
```

The `put_file` method will return the relative path to the file so you can save it to the database and fetch it later.

By default, a file name will be auto generated for you using a UUID4 string. You can specify your own name by using a `name` parameter:

```python
storage.disk('local').put_file('avatars', request.input('avatar'), name="user1")
```

You do not need to specify the extension in the name as the extension will be pulled from the resource object.