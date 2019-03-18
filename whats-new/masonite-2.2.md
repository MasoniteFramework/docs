# Masonite 2.2

{% hint style="danger" %}
This release is still in beta and is not yet released. All information in this documentation section is subject to change.
{% endhint %}

##  Route Prefixes

Previously you had to append all routes with a `/` character.  This would look something like:

```python
Get('/some/url')
```

You can know optionally prefix this without a `/` character:

```python
Get('some/url')
```

## URL parameters can now optionally be retrieved from the controller definition

Previously we ha

```python
def show(self, view: View, request: Request):
    user = User.find(request.param('user_id'))
    return view.render('some.template', {'user': user})
```



