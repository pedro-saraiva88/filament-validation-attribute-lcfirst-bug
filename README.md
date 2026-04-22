# Filament Validation Attribute `Str::lcfirst()` Bug — Reproduction

This repository demonstrates a bug in `filament/forms` where validation error messages
incorrectly lowercase **only the first character** of a multi-word field label.

## Bug Description

When a `TextInput` has a multi-word label such as `"Contact Method"`, Filament calls
`Str::lcfirst()` on the label to build the validation attribute name, producing
`"contact Method"` instead of `"contact method"`.

### Expected

```
The contact method field is required.
```

### Actual

```
The contact Method field is required.
```

## Root Cause

`Filament\Forms\Components\Field::getValidationAttribute()` applies `Str::lcfirst()`
to the label:

```php
// vendor/filament/forms/src/Components/Field.php
public function getValidationAttribute(): string
{
    return $this->validationAttribute ?? Str::lcfirst($this->getLabel());
}
```

`Str::lcfirst('Contact Method')` returns `'contact Method'` — only the first character
is lowercased. Multi-word labels with interior capitals are not handled.

## Steps to Reproduce

```bash
git clone https://github.com/pedro-saraiva88/filament-validation-attribute-lcfirst-bug
cd filament-validation-attribute-lcfirst-bug
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan db:seed
php artisan serve
```

1. Visit `http://localhost:8000/admin`
2. Log in with `admin@example.com` / `password`
3. Go to **Users → Create**
4. Leave "Contact Method" empty and submit
5. Observe the validation message: **"The contact Method field is required."**

## Environment

- PHP 8.2+
- Laravel 11
- Filament v5
- filament/forms v5
