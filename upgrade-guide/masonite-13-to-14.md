# Masonite 1.3 to 1.4

# Introduction

Masonite 1.4 brings several new features and a few new files. This is a very simple upgrade and most of the changes were done in the pip package of Masonite. The upgrade from 1.3 to 1.4 should take less than 10 minutes

## Requirements.txt File

This requirement file has the `masonite>=1.3,<=1.3.99` requirement. This should be changed to `masonite>=1.4,<=1.4.99`. You should also run `pip install --upgrade -r requirements.txt` to upgrade the Masonite pip package.

## New Cache Folder

There is now a new cache folder under `bootstrap/cache` which will be used to store any cached files you use with the new caching feature.

## New Cache and Broadcast Configuration

Masonite 1.4 brings a new `config/cache.py` and `config/broadcast.py` files. These files can be found on the [GitHub](https://github.com/MasoniteFramework/masonite) page and can be copied and pasted into your project.