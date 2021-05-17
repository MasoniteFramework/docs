# Uploading

## Introduction

Very often you will need to upload user images such as a profile image. Masonite let's you handle this very elegantly and allows you to upload to both the disk, and Amazon S3 out of the box. The `UploadProvider` Service Provider is what adds this functionality. Out of the box Masonite supports the `disk` driver which uploads directly to your file system and the `s3` driver which uploads directly to your Amazon S3 bucket.

You may build more drivers if you wish to expand Masonite's capabilities. If you do create your driver, consider making it available on PyPi so others may install it into their project.

Read the "Creating an Email Driver" for more information on how to create drivers. Also look at the `drivers` directory inside the `MasoniteFramework/core` repository.

## Configuration

All uploading configuration settings are inside `config/storage.py`. The settings that pertain to file uploading are just the `DRIVER` and the `DRIVERS` settings.

### DRIVER and DRIVERS Settings

This setting looks like:

```python
DRIVER = os.getenv('STORAGE_DRIVER', 'disk')
```

This defaults to the `disk` driver. The disk driver will upload directly onto the file system. This driver simply needs one setting which is the `location` setting which we can put in the `DRIVERS` dictionary:

```python
DRIVERS = {
    'disk': {
        'location': 'storage/uploads'
    }
}
```

This will upload all images to the `storage/uploads` directory. If you change this directory, make sure the directory exists as Masonite will not create one for you before uploading. Know that the dictionary inside the `DRIVERS` dictionary should pertain to the `DRIVER` you set. For example, to set the `DRIVER` to `s3` it will look like this:

```python
DRIVER = 's3'

DRIVERS = {
    'disk': {
        'location': 'storage/uploads'
    },
    's3': {
        'client': os.getenv('S3_CLIENT', 'AxJz...'),
        'secret': os.getenv('S3_SECRET', 'HkZj...'),
        'bucket': os.getenv('S3_BUCKET', 's3bucket'),
    }
}
```

Some deployment platforms are Ephemeral. This means that either hourly or daily, they will completely clean their file systems which will lead to the deleting of anything you put on the file system after you deployed it. In other words, any user uploads will be wiped. To get around this, you'll need to upload your images to Amazon S3 or other asset hosting services which is why Masonite comes with Amazon S3 capability out of the box.

### Class Based Drivers

You can also explicitly declare the driver as a class:

```python
from masonite.drivers import UploadS3Driver

DRIVER = UploadS3Driver
```

## Uploading

Uploading with masonite is extremely simple. We can use the `Upload` class which is loaded into the container via the `UploadProvider` Service Provider. Whenever a file is uploaded, we can retrieve it using the normal `request.input()` method. This will look something like:

```markup
<html>
    <body>
    <form action="/upload" method="POST" enctype="multipart/form-data">
        <input type="file" name="file_upload">
    </form>
    </body>
</html>
```

And inside our controller we can do:

```python
from masonite import Upload

def upload(self, upload: Upload):
    upload.driver('disk').store(request.input('file_upload'))
```

That's it! We specified the driver we want to use and just uploaded an image to our file system.

This action will return the file name. We could use that to input into our database if we want. All file uploads will convert the filename into a random 25 character string.

```python
upload.driver('disk').store(request.input('file_upload'))
#== '838nd92920sjsn928snaj92gj.png'
```

Lastly, we may can specify a filename directly using the `filename` keyword argument:

```python
upload.driver('disk').store(request.input('file_upload'), filename="username.profile")
#== username.profile.png
```

## Accepting Specific Files

By default, Masonite only allows uploads to accept images for security reasons but you can specify any file type you want to accept by specifying the filetype in the `accept` method before calling the `store` method.

```python
upload.accept('yml', 'zip').store('some.yml')
```

You can also just accept all file types as well:

```python
upload.accept('*').store('some.yml')
```

Note that if you switch the driver or specify a driver, you will need to set the accept after the driver like so:

```python
upload.driver('disk').accept('*').store('some.yml')
```

## Uploading Files

You can upload files directly by passing in a `open()` file:

```python
from masonite import Upload

def upload(self, upload: Upload):
    upload.driver('disk').store(open('some/file.txt'))
```

This will upload a file directly from the file system to wherever it needs to upload to.

### Locations

You can also specify the location you want to upload to. This will default to location specified in the config file but we can change it on the fly:

```python
upload.driver('disk').store(request.input('file_upload'), location='storage/profiles')
```

#### Dot Notation

You can use dot notation to search your driver locations. Take this configuration for example:

```python
DRIVERS = {
    'disk': {
        'location': {
            'uploads': 'storage/uploads',
            'profiles': 'storage/users/profiles',
    }
}
```

and you can use dot notation:

```python
upload.driver('disk').store(request.input('file_upload'), location='disk.profiles')
```

### Uploading to S3

Before you get started with uploading to Amazon S3, you will need the boto3 library:

{% code title="terminal" %}
```text
$ pip install boto3
```
{% endcode %}

Uploading to S3 is exactly the same. Simply add your username, secret key and bucket to the S3 setting:

```python
DRIVER = 's3'

DRIVERS = {
    'disk': {
        'location': 'storage/uploads'
    },
    's3': {
        'client': os.getenv('S3_CLIENT', 'AxJz...'),
        'secret': os.getenv('S3_SECRET', 'HkZj...'),
        'bucket': os.getenv('S3_BUCKET', 's3bucket'),
    }
}
```

{% hint style="warning" %}
Make sure that your user has the permission for uploading to your S3 bucket.
{% endhint %}

Then in our controller:

```python
from masonite import Upload

def upload(self, upload: Upload):
    upload.store(request.input('file_upload'))
```

How the S3 driver currently works is it uploads to your file system using the `disk` driver, and then uploads that file to your Amazon S3 bucket. So do not get rid of the `disk` setting in the `DRIVERS` dictionary.

### Changing Drivers

You can also swap drivers on the fly:

```python
from masonite import Upload

def upload(self, upload: Upload):
    upload.driver('s3').store(request.input('file_upload'))
```

or you can explicitly specify the class:

```python
from masonite.drivers import UploadS3Driver
from masonite import Upload

def upload(self, upload: Upload):
    upload.driver(UploadS3Driver).store(request.input('file_upload'))
```

