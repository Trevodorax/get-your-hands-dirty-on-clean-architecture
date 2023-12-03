# What's wrong with layers?

A layered architecture looks like this:

web -> domain -> persistence

It's a very basic architecture, and works fine if we get it right.

The problem is that it leaves too many doors open for architectural errors.

## It promotes database-driven design

When we create a layered architecture, the domain depends on the persistence, and thus we have to create the persistence before anything else.

As a consequence, we start by building the database schema, and everything is based on the database.

This is bad, because the main part of what we want to implement is the business logic, so everything should be based on this logic (the changes of the data, instead of the persistence of the data).

This often happens when working with an ORM, where the business logic code is mixed to some persistence-related code and highly depends on it.

## It's prone to shortcuts

In a layered architecture, we can't access stuff from other layers.

As a consequence, there is a big temptation to move components to a different layer to allow them to access it.

This often happens with utility functions which end up in the persistence layer. The persistence layer therefore ends up way too big.

The best way to avoid this would be to completely prevent these shortcuts from happening, not by making it forbidden, but by making it impossible.

## It grows hard to test

As the layered architecture gets worse and worse, we start to have persistence code everywhere.

For example, some persistence code may end up it the web layer, for a tiny change in the database.

As this gets bigger, the web layer will get much harder to test, as we will have to mock the persistence before doing the test.

In the end, we'll end up with test code that has a huge quantity of setup because of all the mocks.

## It hides the use cases

As we want to add or modify business logic, we want to be able to easily find stuff in our code.

This is made hard by a layered architecture in 2 ways.

### Vertically

As the business logic is scattered in the 3 layers, it's hard to find existing use cases, and to know where to add new ones.

### Horizontally

In a layered architecture, the with of a layer has no limits.

We therefore often end up with huge services referenced by multiple components of the web layer and referencing multiple components of the persistence layer.

This makes it hard to find where in this huge service is the use case we are looking for.

## It makes parallel work difficult

As the web layer depends on the domain layer which depends on the persistence layer, they have to be developped of after the other.

This can be avoided by creating interfaces, but this prevents us from mixing stuff, which makes it hard to create layered architectures.

Also, as we tend to have big services, it's hard to work together on the same service, which causes even more conflicts when working in parallel.

## How does this help me build maintainable software?

Keeping these traps in mind is always a good thing.

It prevents you from doing them, as the discipline needed to do a layered architecture right is huge.

It also gives solid arguments when trying to advise against taking a shortcut.

Overall, it's a good weapon against these problems when building software, in a layered architecture as well as in other architecture types.
