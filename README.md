# Input Sanitizer

A standalone input sanitizer with options to easily use it in Laravel's FormRequest. It provides a good selection of sanitizers for most use cases, from trimming and lowercase, to masking and casting.

What's the use you're asking? User input is quite arbitrary! Even with guidelines in place, you can't expect a user to enter the appropriate information in the way you hope them to. Validation makes sure the input format is correct, but it shouldn't stop there. An email may need to be trimmed of whitespace, a name may need to be capitalized, and so on. Instead of doing all that manually, Sanitizer will make it a breeze.

## Installation

- Add the package to your composer.json file and run `composer update`:
```json
{
    "require": {
        "fadion/sanitizer": "~1.0"
    }
}
```

Laravel users can add the ServiceProvider and Facade for convenience. This step isn't needed for FormRequest.

- Add `Fadion\Sanitizer\SanitizerServiceProvider::class` to your `config/app.php` file, inside the `providers` array.
- Add a new alias: `'Sanitizer' => Fadion\Sanitizer\Facades\Sanitizer::class` to your `config/app.php` file, inside the `aliases` array.

## Usage

You'll need an array of inputs to be sanitized, most probably from a request object or plain POST/GET data. A basic example would be like the following:

```php
use Fadion\Sanitizer\Sanitizer;

$inputs = ['name' => 'John', 'age' => 31];
$sanitizers = ['name' => 'trim', 'age' => 'int'];

$sanitizer = new Sanitizer;
$newInputs = $sanitizer->run($inputs, $sanitizers);
```

The array of sanitizers needs the input name as the key and a list of sanitizers as either an array, or a string separated with a pipe (Laravel's validator style).

Both of these are valid:

```php
$sanitizersOne = [
    'name' => ['trim', 'ucfirst'],
    'age' => 'int'
];

$sanitizersSecond = [
    'name' => 'trim|ucfirst',
    'age' => 'int'
];
```

For those cases when you need to run some custom code on an input, you can pass a closure as a sanitizer. Don't confuse closure sanitizers with custom sanitizers; these are one-time, anonymous sanitizers that can't be reused. As with the predefined ones, they can be combined with other sanitizers. Take a look at the following code:

```php
$sanitizers = [
    'name' => ['trim', function($value) {
        return str_replace('John', 'Will', $value);
    }],
    'age' => function($value) {
        return $value + 100;
    }
];
```

A few sanitizers accept arguments, such as `date`, `number_format`, `limit`, `mask` and `int`. The syntax is described below:

```php
$sanitizers = [
    'expire' => 'date:m/d/Y',
    'number' => 'number_format:3',
    'body' => 'limit:100',
    'credit_card' => 'mask:+'
];
```

## Usage in Laravel

In Laravel you can directly use the Facade if that's your style. Just remember to include the ServiceProvider and Facade as in the instructions. After that, it's as easy as writing:

```php
$inputs = [/* some inputs */];
$sanitizers = [/* some sanitizers */];

$newInputs = Sanitizer::run($inputs, $sanitizers);
```

## Usage in FormRequest

Form requests are, in my opinion, a very good way of writing validation code. In addition to validation, you can use Sanitizer to clean and transform your inputs in a very intuitive way.

You already know how to create a FormRequest and add rules to it. Let's see how to sanitize those inputs:

```php
namespace App\Http\Requests;

use Fadion\Sanitizer\FormRequest\Sanitizable;

class UserRequest extends Request
{
    use Sanitizable;

    public function rules()
    {
        return [
            'name' => 'required',
            'email' => 'required|email',
        ];
    }

    public function sanitizers()
    {
        return [
            'name' => 'ucwords',
            'email' => 'trim|lower'
        ];
    }
}
```

You'll notice I've used the `Sanitizable` trait! That's what includes the needed functionality and will enable to define sanitizers. You'll also notice there's a `sanitizers()` method. Just return an array with the inputs as keys and Sanitizer will do its job automatically.

## Custom Sanitizers

For most, the predefined sanitizers are all they'll ever need. However, in some rare occassion you may need to run some specific code more than once and it may come in handy to register it as a custom sanitizer. Just remember that complicated logic doesn't belong to the Sanitizer or a FormRequest. Use these only for simple, trival transformations.

### Standalone

Call the static register method anywhere in your code, but before the sanitizer is to be used.

```php
\Fadion\Sanitizer\Sanitizer::register('upper_some', function($value) {
    $some = mt_rand(1, strlen($value) - 1);
    return strtoupper(substr($value, 0, $some)).substr($value, $some);
});
```

### Laravel

The best place to register a custom sanitizer would be in a Service Provider. It can either be a custom, dedicated provider (ie: SanitizerServiceProvider) or in the more general `AppServiceProvider`. Whatever you choose, write your code inside the `boot()` method:

```php
// some service provider
public function boot()
{
    \Fadion\Sanitizer\Sanitizer::register('upper_some', function($value) {
        $some = mt_rand(1, strlen($value) - 1);
        return strtoupper(substr($value, 0, $some)).substr($value, $some);
    });
}
```

The register method accepts a name and a closure. You can register as many custom sanitizers as you need and even override the predefined ones. Now you can use the newly created sanitizer everywhere in your code:

```php
$sanitizers = [
    'name' => 'trim|upper_some'
];
```

## Available Filters

There are more than 30 available filters and most of them use native PHP functions. Refer to the `Filters.php` file for the whole list. Every method is documented, but the code is pretty straightforward for everyone to understand.
