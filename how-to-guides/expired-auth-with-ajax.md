# Handling AJAX requests with expired authentication

## The Problem:
When an ajax request is made from a page that requires a valid user (in the session), the page does not 
redirect (to a Login page) when the auth session has expired.
This makes the page look unresponsive and provides a less than ideal user experience.

## The Solution:
The solution is quite simple though not that obvious.
It has 2 parts:
- The middleware will recognise the expired auth and that the request is ajax, then send a custom `Response` the page will recognise.
- The Javascript on the page will check the ajax response before it gets processes by other event handlres.
If the response matches the criteria the browser redirects to the login page.

## The Code:
### Auth Middleware
In your `AuthenticationMiddleware`    
do something similar to the following
```python
def before(self, request, response):
    if not request.user():
        if request.is_ajax():
            response.content = "MASONITE_LOGIN_REQUIRED"
            response.status(403)
            return response

        return response.redirect(name="login")

    return request
```

### Page Template
Add the `.ajaxComplete()` block the `document.ready` javascript function in your page  
```javascript
$(document).ready(function() {
    $(document).ajaxComplete(
        function (event, request, options) {
            if (request.responseText === "MASONITE_LOGIN_REQUIRED") {
                window.location.href = "{{ route(name='login') }}";
            }
        }
    )
});
```
This is for JQuery but you can adjust this id for whatever JS framework you're using.

## Middleware ordering
This adjustment is not strictly required but does provide a better user experience.

By default the `VerifyCsrfToken` middleware is set as part of the `web` middleware group.
This will prevent the auth middleware checking the request before processing the page form data
and may generate an invalid form error which is not what the user might expect. 

To resolve this you can easily rearrange the order of middlewares.

In your `Kernel.py` file adjust the `route_middleware` similar to this:
```python
route_middleware = {
    "web": [SessionMiddleware, LoadUserMiddleware],
    "auth": [AuthenticationMiddleware, VerifyCsrfToken],
...
}
```
**NOTE:** the `VerifyCsrfToken` comes *AFTER* the `AuthenticationMiddleware`. 
This means that in this example only the routes tagged as `auth` will check the csrf token 
in form data.

## Notes:
- The `MASONITE_LOGIN_REQUIRED` can be anything you like, so making it a `const` of some kind is probably advisable to 
prevent issues with typos.
- The above javascript assumes you have a route named `login` 
