# What is it?
I won't bother to say why microservices are of great importance. Once we begin to use microservices and our architecture becomes comprised of dozens of microservices. Then one question follows: how do we secure our microservices?

Basically there are two typical way to implement this:

+ Decentralized: each microservice implements its own secure logic
+ Centralized: a central service controls all secure issues for other microservices

Obviously, the decentralized one is not scalable and all microservices have to duplicate the same boilerplate code.

# How
## Whitelists & Access Limiting
One basic secure problem is to limit access to a microservice. This can easily be implemented via some reverse proxy, e.g. [Nginx](https://www.nginx.com/resources/admin-guide/restricting-access/).

## Authentication & Authorization
You may first read [this blog](How To Control User Identity Within Microservices).

Basically, the client request for an access token from a central server. When he want to access a service, he sends a request with the token. The central server authenticates it and forwards the request with an OpenID together with some basic user information. The requested service can use the OpenID for identication. It can further request for other information using the OpenID. 

![Demo image](http://nordicapis.com/wp-content/uploads/token_translation_microservices.png)