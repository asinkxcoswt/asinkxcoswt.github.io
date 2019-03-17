---
layout: post
title:  "Domain Behavior Isolation - exploration of extream use of Mixin pattern in domain design for reusability"
categories: Software Design
image: /public/images/domain-behavior-monkey-spiderman.jpg
---

> ... as a monkey developer, you are assigned a task of a spiderman. You are confronted with many possible variation of design decision that can lead to the same functional solution. You have the power to choose one, but with the great responsibility that may lead the future of the software to be in peace or disaster.

# Call for a software design approach for novice developer.

Imagine you are an inexperience developer working for a software development company, your boss assigns you to design and develop a core engine for a software project. He hope that your work will become a profit for the company in future when the engine is reused in other projects. You, however, don't have any idea about what makes a code qualified to be in the *core* engine, and don't know which parts of it will actually be reusable. Even if you know and can lay out a naive version of the engine, the real requirement from the customer will keep demanding super-natural features that tear your design into shreds. You have no choice but to keep dragging the code through your customer's demand and give up your boss's expectation.

I have learned from my recent mistake that a reusable software artifact is naturally created when the similar funtionalities repeats in a hand of a developer more than 1 time, trying to create reusable software (i.e. doing more than what the requirement asks) on the first attempt often results in a premature design. Two points make a direct line. If you give the developer only 1 chances and ask him to design for the future, the design can go in any direction, and is subject to be reworked.

Does this mean that we have to create a junk software once before gaining enough experience to make a good one? I don't like this idea. So I am exploring a design approach that does not demand too much developer's experience, but can help keep our software fairly reusable, maintainable, and is tolerent of our everyday naive design decision. The baseline is that the approach should allow us to design as minimal as possible to fulfill the requirement, meanwhile make the code eligible for reuse and mantainable.

# Don't treat a monkey as a spiderman.

![monkey-spiderman](/public/images/domain-behavior-monkey-spiderman.jpg)

There are already some great software design approaches out there. One of the most important is Domain Driven Design. It introduces concepts and variety of approches that surely will help us to build a highly maintainable and reusable software if we do it correctly. But in some situation, it is not possible to be prepared enough to make a good use of it.

Suppose you have a coconut farm. And you hire monkeys to reap coconuts. The monkeys are not smart enough to devise effective methods, but they can make it through, though a little bit clumsy. So you buy a high-tech machine hoping to help the them and improve the havest process. The monkeys do not fully understand how to operate the machine, they remove parts from the machine and utilize them in their own ways. They then tell you that they are using the high-tech machine. However, the process is as clumsy as always.

We novice developers are like monkeys and DDD is like the high-tech machine in this example. Even experienced developers working together can form a group of monkeys in some situation. Domain Driven Design requires the team to be unified enough that the developers talk the same language as the designers, SA and BA. Without intensive code review sessions that everyone take part in, an individual developer become a spiderman with the great power and responsibility to keep or destroy this unity. Does your project have time enough for this?

Let's see a real world example of how a monkey becomes a spiderman.

Suppose our software has existing `product` domain. Then the requirement comes that ask us to restrict users of a certain role to post products in a certain category, which should be configurable.

Suppose that there are not so much details can be clarified in the specification about which part of the code exactly should be changed, so it is up to the developer to decide. Yet to mention that the due date for the implementation is so tight.

Well, in this situation, as a monkey developer, you are assigned a task of a spiderman. You are confronted with many possible variation of design decision that can lead to the same functional solution. You have the power to choose one, but with the great responsibility that may lead the future of the software to be in peace or disaster.

What does a monkey usually do in this situation? He can enhance something in the product domain entity but he cannot know for sure if it will make sense in long-term. The domain entity is a fundamental element that changes in it will have high impact on the software in overall. So it is natural for the monkey to avoid touching the domain entity. How about creating a new domain service for this? But the monkey inspect the code and see that some implementation classes of existing services have a lot of usefull private methods already, why bother create a new one?. At the end, he choose the easiest way to introduce a new method in some existing domain service that he deems most relevant. Some more dependencies will be added to the service such as `UserService` to see the role of a user, `CategoryService` to see if a category is allowed for a user role and `ApplicationSettingService` to make the logic configurable.

This kind of process repeats through cycles of developmemnt. Later, when we look at the software, we will find that the domain entities become anemic domain models and all the works are put in services, which is highly coupling points in the software and new feature development slowdowns in every cycle due to the complexity in the service.

# Don't try to be a fortune teller.

The previous section talked about maintance problem. In this section, we will talk about reusablity problem.

Let's see some example. Supposed you are designing a domain entity, `Product`, that represents a salable unit in an e-commerce system. At the early stage, you don't know specifically what the customer would like their products to behave, the design team is in the process talking to the customer but you have to start designing and implementing before the process is finished. So you go reseaching existing e-commerce systems out there and see what are common features, structure and behavior for their product management. Then you end up with the first version of `Product` that you deem most common and minimal of what it mean to be a salable unit.

> Note that we present the design using Java interfaces and omit some setters for simplicity.

```java
interface Product {
    Money getPrice();
    String getName(); 
    Integer getQuantityInStock();
    Category getCategory();
}
```

```java
interface Category {
    String getName();
}
```

In this version, you have yet to worry about reusability. It seems to have nothing specific to the customer's requirement. And we can use this design in similar projects in future.

Later, the requirement comes that says a `Product` should have status `ACTIVE` and `INACTIVE` as well as the `Category`. And a `Product` is avaialble to the buyer only if the `Product` and its `Category` have status `ACTIVE` together. So the second version become as follows.

```java
enum ProductStatus {
    ACTIVE, INACTIVE
}
```

```java
enum CategoryStatus {
    ACTIVE, INACTIVE
}
```

```java
interface Category {
    String getName();
    CategoryStatus getStatus();
}
```

```java
interface Product {
    Money getPrice();
    String getName(); 
    Integer getQuantityInStock();
    Category getCategory();
    ProductStatus getStatus();
    default boolean isAvailable() {
        return getStatus().equals(ACTIVE) && getCategory().getStatus().equals(ACTIVE);
    }
}
```

You can see that the simple common design at first begins to have specific requirement mixed in. This is just a simple example, the real requirement will be much more complex than this. But just this simple one, you can start to doubt if your design can be actually reused in other projects? What if the future project has different statuses for products and categories than just `ACTIVE` and `INACTIVE` and the logic to determin availability is not as simple as this? What about if the future customers want to customize their own status? 

you may argue that it's easy to just fork the project into another repository and modify the code to satisfy the requirement of the new project. It will not be an easy job as you might think. All the code of the current project from now on will be designed and implemented based on the assumption that the statuses are only `ACTIVE` and `INACTIVE`. When the code goes big, changing some fundamental assumption affects the whole system. Sometimes, it is easier to just start a new design from scratch.

As a briliant developer, you might think of designing for future by make it flexible as much as possible. For example, you may get all available statuses of an entity from a database table instead of the hard-coded enum. In addition, the table will also has a column to tell wheter each status should be regarded as active or not. Here is such design.

Table `entity_statuses`

| entity_code | status    | active |
|-------------|-----------|--------|
| PRODUCT     | PUBLISHED | true   |
| PRODUCT     | HIDDEN    | true   |
| PRODUCT     | DRAFT     | false  |
| CATEGORY    | ACTIVE    | true   |
| CATEGORY    | INACTIVE  | false  |

```java
interface EntityStatus {
    boolean isActive();
}
```

```java
interface Product {
    Money getPrice();
    String getName(); 
    Integer getQuantityInStock();
    Category getCategory();
    EntityStatus getStatus();
    default boolean isAvailable() {
        return getStatus().isActive() && getCategory().getStatus().isActive();
    }
}
```

```java
interface Category {
    String getName();
    EntityStatus getStatus();
}
```

Your design looks good and you are proud of it. It is very flexible. But with some problems:

1. Your work is delayed because you spend too much time designing and testing this abstraction.
2. You may realise at the end of the project that the final system only need status `ACTIVE` and `INACTIVE`, so your attempt is a total waste of time.
3. It adds uneccessary complexity to the software which slowdowns the subsequent development and maintanance process.
4. You hope to use your design in the next project. But when it comes, it demands much more complex structure than what you have desinged. So you end up inventing a new design for the new project. 

This is an example of an attempt to become a fortune teller which result in a premature design. Instead of designing things straight forward to support the requirement as much as it needs, you try to foresee the future and create the more abstraction of what is really needed. When the true requirment evolves in other directions from the future you saw, you have a hard time revamp the design or bent the existing design in an unreasonable way because the deadline is approaching.

# Reusable units.

We see from the previous sections that domain entities usually evolves to be specific to the current requirement and it is wasteful to try to over-engineer it. We also see that domain service is a point of eventual high coupling in the handle of a spiderman developer. This means the domain entities and services are not eligible to be reused. But our non-trivial works are in these layers.

In this section, we are exploring the possiblity to apply Mixin pattern to help about this problem. The idea is to isolate domain behaviors from an entity and put them in mixins, one behavior per mixin. These mixins then become the reusable units. If you still don't know what is a mixin, you can keep on reading. We will go through an example and conclusion that can clarify it.

From the previous section, we talked about how to improve our first version of `Product` entity to support requirement about the availability. This availability involves some business logic and we hope that we can reuse it in future, but not to over-engineer it. This is an example of a domain behavior that we can put into a mixin.

First we create a Java intefaces called `AvailablititySupport` and `HierachicalAvailabilitySupport<T>`

```java
interface AvailabilitySupport {
    Boolean isAvailable();
}
```

```java
interface HierachicalAvailabilitySupport<PARENT extends AvailabilitySupport> extends AvailabilitySupport {
    PARENT getAvailabilityParent();
    
    default Boolean isHierarchyAvailable() {
        if (!isAvailable()) {
            return false;
        }

        PARENT parent = getAvailabilityParent();
        if (parent instanceOf HierachicalAvailabilitySupport) {
            return ((HierachicalAvailabilitySupport) parent).isHierarchyAvailable();
        } else {
            return parent.isAvailable();
        }
    }
}
```

Any entity that implements `AvailablititySupport` will have to override the method `Boolean isAvailable();` to tell if it is available. With this approach, we don't have to concern how each entity defines it own availability. `Product` entity can maintain its own statuses however it likes. Only thing we want to know is that it tell us whether it is available or not.

The interface `HierachicalAvailabilitySupport` is our mixin. It is required that the implemented entity also implements `AvailablititySupport` because it wants to know if the entity is avaialable or not, but it does care the detailed implementation of how the entity maintain its availability. It also has a generic parameter `PARENT` which also is required to be implemented with `AvailablititySupport` because it also want to know the availability of the parent entity.

The `HierachicalAvailabilitySupport` has a default method `Boolean isHierarchyAvailable()`. It is the functionality **offered** by this mixin. It check availablities of the implemented entity and its parents and return `true` only if all the entities in hierarchy are available. On the other hand, the method `PARENT getAvailabilityParent();` is the information that the mixin **requests** from the implemented entity.

Let's see how it works with our `Product` and `Category`.

```java
interface Category extends AvailabilitySupport {
    String getName();
    CategoryStatus getStatus();

    @Override
    default Boolean isAvailable() {
        return getStatus().equals(ACTIVE);
    }
}
```

```java
interface Product extends HierachicalAvailabilitySupport<Category> {
    Money getPrice();
    String getName();
    Integer getQuantityInStock();
    ProductStatus getStatus();
    Category getCategory();
    
    @Override
    default Boolean isAvailable() {
        return getStatus().equals(ACTIVE);
    }

    @Override
    default Category getAvailabilityParent() {
        return getCategory();
    }
}
```

From the code, we add `AvailabilitySupport` to the `Category` and implement business logic for the method `Boolean isAvailable()`. Now in the `Product` we apply `HierachicalAvailabilitySupport<Category>` mixin to it. This mixin **requests** 2 pieces of information: `Category getAvailabilityParent()` and `Boolean isAvailable()` and **offer** a functionality `Boolean isHierarchyAvailable()` to the `Product` class. In the application layer, we then can determine if a product is avialable to buy as follows

```java
Product product = ... // get product from somewhere
if (product.isHierarchyAvailable()) {
    // do something
}
```

What happen here is that you isolate some of your work into a reusable unit. Now you have someting that is truely reusable: the mixin `HierachicalAvailabilitySupport`.

Let's conclude some terminologies. 

The `AvailabilitySupport` is an example of a *trait*. It is a contract that says the implemented class is *suitable to be applied for a behavior*.
The `HierachicalAvailabilitySupport` is an example of a *mixin*, it is a trait that *add behaviors* to the implemented class.
The entity class acquires a behavior **offered** by a mixin just by implementing the mixin interface. It may have to implement some methods to provide information **requested** by the mixin.

Both trait and mixin are valid reusable units. However, the trait by itself may not be interesting enough to be reused in different projects.

Mixin, on the other hand, can help you a lot. By adding it to your desire class, you got the already tested functionality in a flash. They are worth put in a library to be reuse.

# Spring CGlib Mixin - The Advanced Mixin

As you can see from the previous section that if you isolate a functionality of a class into a trait or mixin, the functionality become reusable. But mixin in Java is quite limited, you can only put some not so complex code into the default method in the interface. What you cannot, and should not, do is for example 

- Query a database, file system or calling external APIs.
- Inject some beans and use their functionalities.
- Keep internal state in a instance variable.

To make our mixin practical, we need more advanced technique than this layman mixin of Java 8. I have already play around with Spring CGlib Mixin and create a library that can help in creating advanced mixins that can leverage all the power of Spring bean. The source code and some example usage is on my Github: [https://github.com/asinkxcoswt/domain-collections](https://github.com/asinkxcoswt/domain-collections).

In the later chapter, I will explain how the core of the library works with Spring CGlib.

# Domain Behavior Isolation Pattern - the extream use of Mixins for domain design

This section is the primary point of this article. I am exploring the chance that we could use Mixin pattern extreamly in our domain design, to the point that we eliminate the use of services in our domain (because we monkeys usually mess up with the service), and leave the domain entities as clean as possible. To be honest, I currently don't know how far it can goes. Maybe, someone has already tired or there already are some approaches similar to or better than this. Anyways, I am on a journey to discover the answer.

![Diagram](/public/images/domain-behavior-diagram.jpg)

Following are some rules for this pattern.

1. Isolate domain behaviors to mixins as much as possible, even if it seems to be deeply intrinsic to the entity. E.g. It is possible to have a mixin called `Orderness` to provide behaviors for entity `Order`. The `Order` class is not reusable as it will evolve by specific requirements. The `Orderness` on the other hand keep what you deem what should be most common for orders in this world regardless to a specific requirement. In the end you reuse `Orderness`, not the `Order`. Becareful that the more functionalities you add into `Orderness` the less it can be reusable in different situation, so it is safer to divide it into small mixins that make up the `Orderness` so that you don't have to re-create the whole `Orderness` when it is a little bit off from a new requirement.
2. Don't use domain service.
3. Design the domain entity and the mixin as minimal as it needs to fulfill the requirement.
4. Put all mixins in separated project, so that it will be reusable in other projects.
5. A mixin should have a single responsibility.

The subsequent sections explore possible benefits of this pattern.

# Declarative Domain Design

Mixins make you class *declarative*. When someone see your class, they know what is it made of and what can it do just by reading the interfaces it implements.

This declarative feature not only good for developers, but also for communication between design team and the developers. It can be what is called Ubiqutious langauge according to DDD. When a new change comes in the right language that refer to traits or mixins, developers know immediately where in the code to be changed, even if he is not the original implementer of the code.

# Plugin & Refactoring

When a change comes that conflicts the existing trait or mixin, You can create a new version of the mixin, remove the old one from the class and plug in the new one. Your entity class is clean because it is not flood with a lot version of logics. Your old mixin still can be reuse in other situation or in other projects. None of the work is wasting.

# An architecture friendly for work distribution.

When a new requirement comes that ask to add a new behavior to a domain, the developer know immediately that he just have to create a new mixin. The mixin pattern when applied this way is friendly to distributed work: each developer is assigned to develop or fix a mixin at a time. Code conflict is reduced. And the team doesn't have to fear if one developer can mess up the entire code for a domain becuase what have been done in each task is limited in mixin scope.  This can help developer work faster, they don't have to waste time reading the entire code base to plan where to put the new change into.

# Naive Design Tolerance - We don't have to become a spiderman anymore

As explained in section [Don't treat a monkey as a spiderman](#don't-treat-a-monkey-as-a-spiderman), when an individual developer can make changes that impacts overall design direction, it will eventually become a mess. With mixin approach, the developer don't have to fear about introducing a naive design decision into the code because the impact only limits to the mixin. And it is easily be improved just by refactoring it with a new mixin.


