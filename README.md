# Forms for Laravel

## Table of contents

1. [Installation](#installation)
2. [Usage](#usage)
3. Forms
    - Instanciation & dependency injection
    - Validation
    - Accessing & processing data
    - State
    - Failure feedback
    - Success feedback
4. Fields
    - Common methods
    - Validation
5. Customization
    - Overwriting existing fields
    - Creating custom fields
6. Digging deeper
    - Creating Form components
    - Generated forms (database-defined forms)
    - Front-end frameworks integration

## Installation

```bash
composer require whitecube/laravel-forms
```

This package will auto-register its service provider.

## Usage

Every form's starting point is its controller class. To get started, create a new Form class like the one in the example below. You can store this file wherever you want inside your application's namespace.

```php
namespace App\Forms;

use Whitecube\Forms\Form;

class ContactForm extends Form
{
    public function fields(): array
    {
        return [];
    }
}
```

Next, let's define the form's fields. This package ships with a few common field types, such as `Text`, `Textarea`, `Email`, `Phone`, `Checkbox`, `Checkboxes`, `Radios`, `Select`. However, you can overwrite these fields' behavior or create your own! For more information, take a look at the custom fields section below.

```php
namespace App\Forms;

use Whitecube\Forms\Form;
use Whitecube\Forms\Fields\Text;
use Whitecube\Forms\Fields\Email;
use Whitecube\Forms\Fields\Textarea;

class ContactForm extends Form
{
    public function fields(): array
    {
        return [
            Text::make('firstname', 'Your firstname'),
            Text::make('lastname', 'Your lastname'),
            Email::make('email', 'Your e-mail'),
            Textarea::make('message', 'Tell us a story!'),
        ];
    }
}
```

Fields have multiple useful API methods that will allow validation and other configurations.

In order to display the form, we'll have to pass it to a view. Most commonly, this would be done from a Controller class:

```php
namespace App\Http\Controllers;

use Illuminate\View\View;
use App\Forms\ContactForm;
 
class ContactController extends Controller
{
    public function show(): View
    {
        return view('pages.contact', [
            'form' => ContactForm::make()
        ]);
    }
}
```

> [!IMPORTANT]
> Beware that you should always use the static `make` or `makeWith` methods in order to create a Form instance since this is the only way to properly setup the Form's state.

The form can now be used inside a Blade template. Here's an example of a typical form implementation:

```blade
<div class="form">
    @if($form->successful())
        <div class="form__feedback form__feedback--success">
            <p>{{ $form->success('Optional default success message.') }}</p>
        </div>
    @else
        @if($form->failed())
            <div class="form__feedback form__feedback--error">
                <p>{{ $form->error('Optional default "generic" error message.') }}</p>
            </div>
        @endif
        <form action="{{ route('contact') }}" method="POST" class="form__container">
            @csrf

            @foreach($form->fields() as $field)
                <x-form-field :$field />
            @endforeach

            <div class="form__actions">
                <button type="submit" class="form__btn form__btn--submit">Submit</button>
            </div>
        </form>
    @endif
</div>
```

There are a few important things to notice in the example above:

- The provided `$form` object has a few useful state manipulation methods making it easy to display failure & success messages ;
- You still have full control over the form's markup & routing ;
- Fields are rendered using a simple loop and a provided Blade `form-field` component, which you could replace with your own, extend and customize.

> [!TIP]
> Not using Blade? Don't worry, we got you covered! The form objects, their state and fields are fully JSON-serializable and can be used in the front-end framework of your choice.

Now that the form can be submitted, it is time to implement its processing behavior. Let's head back to our `ContactController` class and add a `process` route method:

```php
namespace App\Http\Controllers;

use App\Forms\ContactForm;
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
 
class ContactController extends Controller
{
    // ...

    public function process(Request $request): RedirectResponse
    {
        $form = ContactForm::make()->validate($request);

        if($form->failed()) {
            return back()->withForm($form);
        }

        $data = $form->data();

        // do stuff...

        return back()->withForm($form);
    }
}
```

And that's it. A few noticeable features:

- By default, the request data is validated based on the defined form fields validation rules. However, you can also add authorization/validation hooks to the form itself, providing full control over form-wide verifications that should be executed before or after the fields get validated ;
- The form's state can be preserved after a redirect response thanks to the `withForm` method we added to Laravel's `RedirectResponse` class. This way, the form's state will be flashed to the session and success or failure feedback will be automatically showed upon reload while fields will be filled with their old values ;
- The validated (and eventually processed) form data can be accessed using the form's `data()` method. In order to keep controllers as clean as possible, it is also possible to add extra data processing behavior (such as moving files, value parsing, ...) using specific hooks defined in your Form classes.

---

WIP.

---

## Development Roadmap

WIP.

Need some specific features? Open an issue or a PR and we'll take a look at it! But remember, if you want professional support, don't forget to sponsor us! 🤗

## 🔥 Sponsorships 

If you are reliant on this package in your production applications, consider [sponsoring us](https://github.com/sponsors/whitecube)! It is the best way to help us keep doing what we love to do: making great open source software.

## Contributing

Feel free to suggest changes, ask for new features or fix bugs yourself. We're sure there are still a lot of improvements that could be made, and we would be very happy to merge useful pull requests. Thanks!

## Made with ❤️ for open source

At [Whitecube](https://www.whitecube.be) we use a lot of open source software as part of our daily work.
So when we have an opportunity to give something back, we're super excited!

We hope you will enjoy this small contribution from us and would love to [hear from you](mailto:hello@whitecube.be) if you find it useful in your projects. Follow us on [Twitter](https://twitter.com/whitecube_be) for more updates!