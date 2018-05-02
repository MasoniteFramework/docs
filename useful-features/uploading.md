# Uploading

## Introduction

Very often you will need to upload user images such as a profile image. Masonite let's you handle this very elegantly and allows you to upload to both the disk, and Amazon S3 out of the box. The `UploadProvider` Service Provider is what adds this functionality. Out of the box Masonite supports the `disk` driver which uploads directly to your file system and the `s3` driver which uploads directly to your Amazon S3 bucket.

You may build more drivers if you wish to expand Masonite's capabilities. If you do create your driver, consider making it available on PyPi so others may install it into their project.

Read the "Creating an Email Driver" for more information on how to create drivers. Also look at the `drivers` directory inside the `MasoniteFramework/core` repository.

## Configuration

All uploading configuration settings are inside `config/storage.py`. The settings that pertain to file uploading are just the `DRIVER` and the `DRIVERS` settings.

### Driver and DRIVERS Settings

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

### Uploading

Uploading with masonite is extremely simple. We can use the `Upload` class which is loaded into the container via the `UploadProvider` Service Provider. Whenever a file is uploaded, we can retrieve it using the normal `Request.input()` method. This will look something like:

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
def upload(self, Upload):
    Upload.store(Request.input('file_upload'))
```

That's it! We specified the driver we want to use and just uploaded an image to our file system.

This action will return the file system location. We could use that to input into our database if we want:

```python
>>> Upload.store(Request.input('file_upload'))
storage/uploads/new_upload.png
```

We may also need to get the filename of the upload. If the request input is a file upload, we have some additional attributes we can use:

```python
>>> Request.input('file_upload')
new_upload.png
```

Lastly, we may need to prepend the file name with something like a `uuid` or something or even just a normal string. We can do so by using the `storePrepend()` method:

```python
>>> Upload.store(Request.input('file_upload'), 'prepend_name_')
prepend_name_newupload.png
```

### Uploading to S3

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
def upload(self, UploadManager):
    UploadManager.driver('s3').store(Request.input('file_upload'))
```

How the S3 driver currently works is it uploads to your file system using the `disk` driver, and then uploads that file to your Amazon S3 bucket. So do not get rid of the `disk` setting in the `DRIVERS` dictionary.

