---
layout: post
title:  "How to Structure Django Projects"
date:   2018-06-25 12:00:00 +0100
---

Here are some notes on how to layout a Django project. It breaks away from structuring a project around Django “apps” and instead uses a clear separation between three core layers; data, domain, and interfaces. Let’s use the following example, an e-commerce site called “Crema” where people can purchase coffee goods. Below is a layout of the fundamental directories.

```plaintext
src/
    crema/
        data/
            migrations/
            models/
        domain/
            baskets/
            orders/
            products/
            users/
        interfaces/
            actions/
                management/
                    commands/
            dashboard/
                orders/
                products/
                users/
            store/
                account/
                basket/
                checkout/
                products/
    tests/
        functional/
        unit/
```

The `src/` directory lives in the root of the repository, alongside files such as `README.md`. The `crema/` directory contains all the project files, and all the test files live in the `tests/` directory. Let’s drill down into the three main layers.

## Data

```plaintext
data/
    migrations/
        __init__.py
        0001_initial.py
    models/
        __init__.py
        basket.py
        order.py
        product.py
        user.py
    __init__.py
```

The `data/` directory contains all the model and migration files for the project. Opt for one model class per module and ensure that they’re all imported into `models/__init__.py` so Django can discover them and generate migrations. This also results in one simple and consistent import of the data layer throughout the project to access the models. For example:

```python
>>> from crema.data import models
>>> products = models.Product.objects.all()
```

These model modules should only import other model modules, and possibly some generic utils modules. The model classes should be very lightweight. Their purpose is to perform trivial read and write operations against the database. Business logic should most certainly not live in the data layer, that belongs in domain.

## Domain

The `domain/` directory contains all the business logic files. Each domain area is a directory containing modules of related functionality. Orders, for example, might contain the following; an `engine.py` module with functions related to creating new orders; a `management.py` module for managing existing orders, such as cancelling them; and a `processing.py` module for performing the steps related to dispatching an order.

```plaintext
domain/
    orders/
        __init__.py
        engine.py
        management.py
        processing.py
    __init__.py
```

The key point is that the functions within these modules remain focused on their specific domain and call upon other domain areas for theirs. With this focus, all changes flow through dedicated functions and it becomes easier to add additional operations to them as business needs change. Ultimately, this produces consistent results regardless of which interface is calling into the domain layer.

## Interfaces

The purpose of the interfaces layer is to validate input, call the domain layer, and return output. Similar to the data layer, business logic shouldn’t live in these interfaces, their purpose is to handle I/O.

There’s two HTTP interfaces in this Crema project. One named `store/` that’s public facing, and one named `dashboard/` that’s for company staff (hosted under a separate subdomain). In both interfaces a form validates the input, a view calls the domain layer with this validated input, and then converts the return value (or exception) into the appropriate HTTP response.

A third interface, named `actions/`, is a collection of Django management commands. Similar to the other two interfaces, a `BaseCommand` subclass would use a parser to handle the input, then make a call to the domain layer, and then write some output to `self.stdout`.

These interfaces will import the data layer to retrieve objects from the database. They must not, however, bypass the domain layer and perform any create, update, or delete operations. This means using `ModelForm`, `CreateView`, `UpdateView`, or `DeleteView` is prohibited. Whilst these Django classes can be convenient, they perform operations on models directly instead of calling upon the domain layer.

More interfaces might come and go but they all follow a similar pattern; calling upon the domain layer to perform operations and handle the business logic.

## Tests

The `tests/` directory lives alongside the the `src/` directory. Betters to have all the test files contained in one place rather than sprinkled amongst the project files. The tests should be grouped into unit tests and functional tests. You may want to group integration tests separately too. The test modules within each of these groups are structured into directories that mimic the project layout.

```plaintext
src/
    crema/
        data/
            models/
                order.py
    tests/
        unit/
            data/
                models/
                    test_order.py
```

## Django apps

One last note to add is that this project structure results in fewer `INSTALLED_APPS` entries. The `data/` directory, containing the models, and the interfaces directories, containing templates, static files, management commands, etc., all need to be included as entries. The `domain/` directory, however, won’t need to be included as it doesn’t contain any of these things, and instead is a collection of Python modules that Django doesn’t need to discover or search.

In conclusion, this project structure aims at creating a single and accessible data layer, a consistent domain layer to handle business logic, and an interfaces layer that handles different inputs and outputs to the system.
