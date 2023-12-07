<style>
  .half {
    width: 50%;
  }
</style>

# Implementing a web interface

In most applications, there is a web entrypoint (API/website).

Let's see how we can build one following the hexagonal architecture.

## Dependenxy inversion

Here is what the interactions between the input adapters and the application services look like:

<div class="half">

![in adapter and services](./assets/7.png)

</div>

The web adapter is called an "incomming" or "driving" adapter.

The outside world calls it, and it calls the application core, telling it to do something.

It does so by calling an incomming port, which is implemented by a service.

This is dependency inversion, as the input adapter could call the services directly, as showed below:

<div class="half">

![in adapter and services no ports](./assets/8.png)

</div>

But we won't do that, because having ports gives us a clear definitions of how and where the outisde world interacts with our application core.

This is very useful for maintaining and understanding the project.

Again, Chapter 11 discusses the possibility to take this shortcut.

If we need a two way communication between the web adapter and the application core, we can also make the web adapter both a "driving" (in) and "driven" (out) adapter, like in the graph below:

<div class="half">

![in adapter and services no ports](./assets/9.png)

</div>

In this chapter, we will focus on a web adapter as an incomming adapter only.

## Responsibilities of a web adapter

Here is what the web adapter does, with examples for a REST API adapter:

0. Listen for requests from the outside (listen on a specific URL, for example)
1. Map request to Java objects (HTTP object to Java object)
2. Perform authorization checks (make sure the user is authentified and can access this route)
3. Validate input (checking that the HTTP object can be transformed into the input model of the use case)
4. Map input to the input model of the use case (creating an instance of the input model class)
5. Call the use case (passing the instance we just created)
6. Map output of the use case back to HTTP (Java object to HTTP object)
7. Return HTTP response (send HTTP response back to caller)

This responsibility on the incomming adapter allows the application core to not know we use HTTP.

It will therefore be easy to switch to other adapters later on (GraphQL, etc.).

The distinction between what must be done in the application core and in the web adapter comes naturally if we start by creating the application core, implementing the use cases first.

## Slicing controllers

