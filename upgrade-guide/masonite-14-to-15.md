# Masonite 1.4 to 1.5

# Introduction

Masonite 1.5 doesn't bring many file changes to Masonite so this upgrade is fairly straight forward and should take less than 10 minutes.

## Requirements.txt

All requirements are now gone with the exception of the WSGI server (`waitress`) and the Masonite dependency. You should remove all dependencies and only put:

```
waitress==1.1.0
masonite>=1.5,<=1.5.99
```

## Site Packages Configuration

If you have added your site packages directory to our packages configuration file, you can now remove this because Craft commands can now detect your site packages directory in your virtual environment.

## Api Removal

Remove the `masonite.providers.ApiProvider.ApiProvider` from the `PROVIDERS` list as this has been removed completely in 1.5

If you are using the `Api()` route inside `routes/api.py` for API endpoints then remove this as well. You will need to implement API endpoints using the new Official Masonite Entry package instead.

## Finished

That's it! You have officially upgrades to Masonite 1.5