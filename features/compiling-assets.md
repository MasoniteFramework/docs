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
mix
  .js("resources/js/app.js", "storage/compiled/js")
  .postCss("resources/css/app.css", "storage/compiled/css", [
    //
  ]);
```

This will move these 2 files, `resources/js/app.js` and `resources/css/app.css` and compile them both into the `storage/compiled` directory.

**Feel free to change which directories the files get compiled to. For more information on additional configuration values take a look at the** [**Laravel Mix Documentation**](https://laravel-mix.com/docs/6.0/examples)

### Installing TailwindCSS

Laravel is using Laravel mix so you can just follow guidelines to setup TailwindCSS for Laravel Mix on [TailwindCSS Official Documentation](https://tailwindcss.com/docs/guides/laravel#setting-up-tailwind-css).

### Installing Vue

Please follow the guidelines directly on [Laravel Mix Vue Support Documentation](https://laravel-mix.com/docs/6.0/vue)

## Compiling

Now that we have our compiled assets configured we can now actually compile them.

You can do so by running:

```text
$ npm run dev
```

This will compile the assets and put them in the directories you put in the configuration file.

You can also have NPM wait for changes and recompile when changes are detected in frontend files. This is similiar to an auto reloading server. To do this just run:

```text
$ npm run watch
```

## Versioning

Laravel Mix can take care of file hashing when releasing assets to production, by adding a hash suffix to your assets to
automatically bust cache when loading assets. To enable this you can add the following in your `webpack.mix.js` file:

```js
if (mix.inProduction()) {
  mix.version();
}
```

More information on this feature in [Laravel Mix Versioning](https://laravel-mix.com/docs/6.0/versioning).

After Laravel Mix compiled your assets you won't be able to know the exact filename (because of the hash) you should use in your views to reference your assets. You can use Masonite `mix()` helper to resolve the correct file path to your assets.

```html
<script src="{{ mix('resources/js/app.js') }}"></script>
```
