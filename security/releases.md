# Releases

## Introduction

Masonite takes security seriously and wants full transparency with security vulnerabilities so we document each security release in detail and how to patch or fix it.

# 2.1.2

## Issue

There were 2 issues involved with this release. The first is that input data was not being properly sanitized so there was an XSS vulnerability if the developer returned this directly back from a controller. 

Another issue was that there were no filters on what could be uploaded with the upload feature. The disk driver would allow any file types to be uploaded to include exe, jar files, app files etc.

**There was no reported exploitation of this. These were both proactively caught through analyzing code and possible vulnerabilities.**

## Fix

The fix for the input issue was simply to just escape the input before it is stored in the request dictionary. We used the html core module that ships with Python and created a helper function to be used elsewhere.

The fix to the second issue of file types was to limit all uploads to images unless explicitly stated otherwise through the use of the `accept` method.

## Patch

The patch for this is to simply upgrade to 2.1.3 and explicitly state which file types you need to upload if they are not images like so:

```python
def show(self, upload: Upload):
    upload.accept('exe', 'jar').store('...')
```

Those choices to accept those files should be limited and up to the developer building the application sparingly.

# 2.3.24, 3.0.4

## Session Based CSRF tokens

The security community recently discovered several vulnerabilities from the CSRF features of Masonite. The issue was that Masonite used session based CSRF tokens. Meaning a CSRF token was set in the cookie and then was used for each request. This provides an attacker to use size based sub channels to leak the token 1 character at a time. This is known as a BREACH attack.

The solution to this was to change the CSRF tokens to be changed on every request. Previously there was an attribute set on the CSRF middleware of `every_request`. This came disabled by default but in the security patch, this attribute is ignored and all csrf tokens will regenerate on every request.

## Timing Attacks

Masonite was checking if the csrf token matched the token that was sent in the request. This is fine but we were using a comparison operator like `==`. Python will check each character one at a time and return False once a character does not match. An attacker is able to send an arbitrary string and measure down to the nano seconds on how long the comparison takes. 

The solution to this is to use a constant time comparison check so that comparing 2 strings always takes the amount of time. This was accomplished using `hmac.compare_digest`.

## Cookie Tossing

Cookie tossing is when an attacker is able to obtain access to a subdomain and set cookies on the parent domain. They are then able to set the CSRF token to whatever they want which will match the CSRF token that is submitted.

The solution to this is now Masonite sets a session ID which is checked against the CSRF token and the users session ID which the attacker would not be able to guess. The session ID is unique to each user and changes every 5 minutes.

## Cookie Overflow

Browsers can only store a certain amount of cookies before a browser starts deleting old cookies and replace them with the attackers cookies.

The solution to this is now Masonite sets a session ID which is checked against the CSRF token and the users session ID which the attacker would not be able to guess. The session ID is unique to each user and changes every 5 minutes.

## Checking Unsafe HTTP Methods

We were previously only checking CSRF tokens on POST requests which allowed attackers the ability to change the request method in a form to bypass the CSRF protections. 

The solution is to check the CSRF token for other unsafe request methods like POST, PUT, PATCH and DELETE

## Replay Attacks

If session based CSRF tokens are used, an attacker has the ability to reuse the CSRF token it was able to get to attempt more than 1 attack. This assumes the CSRF token was able to be leaked in the first place.

The solution to this is Masonite now has per request CSRF tokens