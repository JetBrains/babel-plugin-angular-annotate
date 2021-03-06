# babel-plugin-angular-annotate

>  Make angular dependency annotation minification proof

[![Build Status][travis_badge]][travis]

## Compatibility

The version 2.x uses babel 6.x plugin API, for babel 5.x versions use the babel-plugin-angular-annotate 1.x

## Installation

```sh
npm install babel-plugin-angular-annotate
```

## Usage

### Via `.babelrc` (Recommended)

**.babelrc**

```js
{
  "plugins": [
    ["angular-annotate", [configurations...]]
  ]
}
```

### Via CLI

```sh
$ babel --plugins angular-annotate script.js
```

### Via Node API

```javascript
require("babel-core").transform("code", {
  plugins: [
    ["angular-annotate", [configurations...]]
  ]
});
```

## Known issues

- Some injections wont work properly when using this plugin in conjuction with `babel-preset-es2015`. To get it working you need to use `"passPerPreset": true` in your `.babelrc`.

## Configuration

`angular-annotate` accepts a json like injection configuration starting with an array containing two items in this format: `[method call, args]`.

`method call` is expressed as a string with the service name and method call. For instance `"$injector.invoke"`.
You can also nest calls. For instance: `"$httpProvider.interceptors.push"`.

`args` is where you map each param with the corresponding injection strategy. The two possible are: `"$injectFunction"` and `"$injectObject"`.
Any other value will be ignored.

`$injectFunction` will transform:

```js
function (a, b, c) {
}
```

to

```js
['a', 'b', 'c', function (a, b, c) {
}]
```

For instance to create a rule for `$injector.invoke` you can apply the following configuration: `["$injector.invoke", ["$injectFunction"]]`.

So the following will be transformed:

Before:

```js
$injector.invoke(function($state) {
  $state.go('somewhere');
});
```

After:

```js
$injector.invoke(['$state', function($state) {
  $state.go('somewhere');
}]);
```

`$injectObject` will apply `$injectFunction` for each object value. This is mainly used in the `resolve` property from some services. For example:

The `$routeProvider.when` configuration can be expressed with the following:

```json
["$routeProvider.when", ["_", {
  "controller": "$injectFunction",
  "resolve": "$injectObject"
}]];
```

Before:


```js
$routeProvider.when('/foo', {
  controller: function($scope) {
    $scope.message = 'foo';
  },
  templateUrl: 'foo.html',
  resolve: {
    store: function (foo) {
    }
  }
});
```

After:

```js
$routeProvider.when('/foo', {
  controller: ['$scope', function($scope) {
    $scope.message = 'foo';
  }],
  templateUrl: 'foo.html',
  resolve: {
    store: ['foo', function (foo) {
    }]
  }
});
```

Note that since we don't want to do anything in the routeName we use a `"_"` to ignore it.


### Presets

Since configuring each service injection can be tedius, this libray includes some presets like: `"angular", "ngMaterial", "ngRoute" and "ui.router"`.
So you can simple include the following in .babelrc:

```json
{
  "plugins": [
    ["angular-annotate", ["angular", "ngMaterial", "ui.router"]]
  ]
}
```

Check the [main file](./src/index.js) to see what injections are currently handled.

## Running Tests

`npm test`

## Contributing

1. Fork it
1. Create your feature branch (`git checkout -b my-new-feature`)
1. Commit your changes (`git commit -am 'Add some feature'`)
1. Push to the branch (`git push origin my-new-feature`)
1. Create new Pull Request

[travis]: https://travis-ci.org/marcioj/babel-plugin-angular-annotate
[travis_badge]: https://api.travis-ci.org/marcioj/babel-plugin-angular-annotate.svg?branch=master
