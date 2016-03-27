JavaEE is a full-fledged enterprise application framework, presenting a complete example of what an expterprise application should look like. Concept and solutions appeared in JavaEE can be safety applied in other technique stacks.

# What does JavaEE offer?
## Core concpet
First I will present core concept in j2e which offers us a clear and effective way to thinking about our applications.

### Containers
Java2E is generally a multi-tier application: client-tier -> web-tier -> business-tier -> eis-tier. J2E provides common functionality support or underlying services support for all tiers in the form of containers. Application logic are divided into components, which run inside relevant containers. Containers are the interface between a component and the low-level, platform-specific functionality that supports the component.

Containers enable programmers concentrate on core logic and do not need to worry about how a component is created, assemblied and how a component interacts with the external worlds or other components.

### Components
A component self-contained functional software unit assembled into a Java EE Application. Basically there are three kinds of components  
+ Client components: such as applets which are depracated now.
+ Web components: such as servlets and jsps, running inside a web container.
+ Business components: such as ejbs, running inside an ejb container.

Components enable programmers write modular and reusable functional unit, which effectively controls business complexities.

Note: components can reside in different processes which means that J2E is a distributed standard.

# Component management
Components are core concept in J2E. In this section, I will illustrate lifecycles of components and interaction between components, which further demostrate how components accomplish their duties(business logic).

## Inverse of control
Components are managed by containers. Containers will automatical create and assemble a component on demand. When synchronuous interaction is required, a component must retain references to other components, which is achieved via dependency injection. Containers know all interfaces and relevant implementations.

+ Lifecycle control: components can define lifecycles callbacks as well as their scopes via annotations or xmls.
+ Dependency injection: components can define what interfaces they need as well as other resources such as database connection pools.

## Interaction
Components can collabratively fulfill a single task which is achieve by either strong interaction such as direct invokation or weak interaction such as messaging. Components can config their method as a reciever method or a sender method.

## Extending components
Programmers can extend a component via interceptors or so-called AOP techniques. Typicall examples are logging, auditing, and transaction management.

## Scalability
I want to particularly emphasis the scalability of components. 

Components can be distributed and there might be multiple components to fulfill a single functionality. So pay attention to your components, make they scalable(stateless/loosely coupled with other components).

# Underlying services
Now that we have components managed by containers, how can components benefit from containers?

As mentioned above, containers can manage components lifecycles, provide messaging support, provide AOP support, they can further perform the following things:

+ Task scheduling: containers can schedule components and execute certain methods in a proper timepoint or repeatedly.
+ Asynchronuous execution: containers can execute certain methods asynchronuously.
+ Securing invokation of components: containers can authorize invokations to prevent unintentional invokation.
+ Resource management: all components can be looked up via JNDI.
+ Other services:
    + Email services
    + Database connection

Almost all business-free functionalities can be extracted to the service layer and put it into the container which further support other components.

# Application assembly and deployment
Once we have a varity of components, we can assemble them into a full-functional application and put them into containers to run the application. This is often done via configuration files or annotations.

# Conclusion
JavaEE platform is great. Even if we do not use java, the same principles can be applied two. From a broad perspective, say the distributed system, services are generally viewed as `components` and cloud platform as the `container`.
