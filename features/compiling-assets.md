Masonite uses Laravel Mix which provides a really simple way to handle asset compiling even greater than simple SASS and LESS. You don't need to be an expert in either Laravel Mix or NPM to compile assets, though.

## Getting Started

To get started we can simply run NPM install:

```text
$ npm install
```

This will install everything you need to start compiling assets.

## Configuration

The configuration settings will be made inside your `webpack.mix.js` file located in the root of your project.

You can see there is already an example config setup for you that looks like this:

```javascript
mix.js('storage/static/js/app.js', 'storage/compiled/js')
    .sass('storage/static/sass/style.scss', 'storage/compiled/css');
```

This will move these 2 files, `storage/static/js/app.js` and `storage/statis/sass/style.scss` and compile them both into the `storage/compiled` directory.

**Feel free to change which directories the files get compiled to. For more information on additional configuration values take a look at the** [**Laravel Mix Documentation**](https://laravel.com/docs/6.x/mix)

## Compiling

Now that we have our compiled assets configured we can now actually compile them.

You can do so by running:

```text
$ npm run dev
```

This will compile the assets and put them in the directories you put in the configuration file.

You can also have NPM wait for changes and recompile when changes are detected. This is similiar to an auto reloading server. To do this just run:

```text
$ npm run watch
```

