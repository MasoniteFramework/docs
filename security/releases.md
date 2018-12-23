# Releases

## Introduction

Masonite takes security seriously and wants full transparency with security vulnerabilities so we document each security release in detail and how to patch or fix it.

## 2.1.2

### Issue

There were 2 issues involved with this release. The first is that input data was not being properly sanitized so there was an XSS vulnerability if the developer returned this directly back from a controller. 

Another issue was that there were no filters on what could be uploaded with the upload feature. The disk driver would allow any file types to be uploaded to include exe, jar files, app files etc.

**There was no reported exploitation of this. These were both proactively caught through analyzing code and possible vulnerabilities.**

### Fix

The fix for the input issue was simply to just escape the input before it is stored in the request dictionary. We used the html core module that ships with Python and created a helper function to be used elsewhere.

The fix to the second issue of file types was to limit all uploads to images unless explicitly stated otherwise through the use of the `accept` method.

### Patch

The patch for this is to simply upgrade to 2.1.3 and explicitly state which file types you need to upload if they are not images like so:

```python
def show(self, upload: Upload):
    upload.accept('exe', 'jar').store('...')
```

Those choices to accept those files should be limited and up to the developer building the application sparingly.

