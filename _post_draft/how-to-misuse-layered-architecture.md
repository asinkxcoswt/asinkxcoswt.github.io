# Some Problems About Layered Architecture in Classic Spring Application

Most developers are familiar with layered architecture. It is a level-1 sword to start off our software design adventure. We see it in many online tutorials out there. Mark Richards begins his book [Software Architecture Patterns](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html) with this architecture pattern. Petri Kainulainen has an [article](https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/) that summarize what layered architecture looks like in a classic Spring web application. Let's borrow one of his diagram here.

(the diagram)

Theoritically, applications should have at least following benefit using this architecture.

**Separation of Concern through Layer Isolation** 2 layers talk to each other via interfaces. We can change implementation in a layer without affecting other layers because the interfaces are not chagned.
**Low Coupling** a layer depends other layers by interfaces.
**High Testability** we can mock the funcationlity of a layer in testing so that the test does not depend on the other dependent layers.

When we are not sure that we have enough knowledge and skills to design an application in more advanced architecture, we tend to resort to this classic layered architecture or "dance around" with it, avoid making the proper analysis. Sometimes, we use it just so that we can tell our customers that we use some architecture rather than none. When this happen, we keep finding ourself wondering why our application doesn't have the aforementioned advantages of the architecture.

In this article, I will talk about examples of how to improperly apply layered architecture that discard its benefits.**

# Have you ever ask what kind of concerns that can benefit from being separated across layers?

Johannes Brodwall has interesting article discussing about different in layering of networking stack (HTTP, TCP, IP, and so on) and enterprise applications. In networking, we separate work into layers, the higher layer can use the lower one without having to concern about its implementation details. The architecture works very well in this situation, as it is less often to have changes in the lower levels. Like networking, enterprise applications have works that seems to fit to be separated into layers such as database manipulation, business logic and presentation logic. Unlike networking, changes in enterprise applications often affect all application layers, from database to presentation.

With this nature of enterprise applications, layers tend to obstruct the way we actually work. To change something, a developer has to go through unecessary layers to make thing happen. Each layer has price, developers will need more time to understand the code and make changes, E2E test between layers is always expected by practice. And, most importantly, we have to sacrifice some Object Oriented capabilities.

To see what I mean, let's suppose we are developing a REST API using Spring Boot. Spring practitioners often divide his application into 3 layers according to Spring's predefined componenet archetypes: `@RestController`, `@Service` and `@Repository`. The controller talks to the service through an interface. The service, in turn, talks to the repository through an the repository interface.

```
@RestController
class ProductController {

    private ProductSerivce productService;

    @Autowire
    public ProductController(ProductSerivce productService) {
        this.productService = productService;
    }

    @GetMapping("/products/{productUuid}")
    public ProductDTO createProduct(UUID productUuid) {
        Product product = productService.findProductByUuid(productUuid);
        ProductDTO productDto = ProductDTO.map(product);
        return productDto;
    }
}
```

```
@Service
interface ProductService {
    Product findProductByUuid(UUID productUuid);
}
```

```
class ProductServiceImpl implements ProductService {

    private ProductRepository productRepository;

    @Autowire
    public ProductServiceImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public Product findProductByUuid(UUID productUuid) {
        return productRepository.findByUuid(productUuid);
    }
}
```

```
@Repository
interface ProductRepository implement JpaRepository<UUID, Product> {
    Product findProductByUuid(UUID productUuid);
}
```

There are 2 code smells in the above example. First, since the controller cannot access repository directly, it delegates the work to the service. The service, in turn, just call to the repository without additional logic. This is called **Sinkhole anti-pattern**. Another smell is that we have something call `ProductServiceImpl` which has been named with the suffic `impl` just so that the name does not conflict with the interface name `ProductService`. This indicate that our **actual OO design** doesn't have this inheritance relation, it exists just to make up the **layer**. This abuse of the OO interface is called **Single Implementation anti-pattern**.

Just to conform to the layered architecture design, we add 2 kinds of anti-pattern to our code. It can be fine if it's worth it. Of course, it's not like we have done this for no reason. Theoritically, we are supposed to have the benefit of layer isolation.

> The layers of isolation concept means that changes made in one layer of the architecture generally don’t impact or affect components in other layers.

In layered architecture, you can swap out or alter an implementation of a component without affecting other layers. But as discussed in the beginning of the section, changes in enterprise application usually affect all layers. It's very rare that changing or swaping soem concrete classes can fulfill a requirement. In the most case, we have to change database query and alter the intefaces of all layers to make the change available to users.

To conclude this section, we see an example of how misplacing layered architecture can make our life harder, but it doesn't mean we should not use the architecture. In my opinion, there are some valid situations to separate application into layers, such as

- When a layer can exist and be useful with out the higher layers.
- When we have 2 separate team working in parallel for each layer.

# Layering VS Component Classification

In the previous section, we have discussed about how our application can be simplified and our OO disign can be cleaner just by removing unnecessary layers. However, this does not mean we should do database manipulation logic in a controller or something like that. Separation of Concern is still a pilar of software design.

We have mentioned that Spring has predefined component archetypes: `@RestController`, `@Service` and `@Repository`. Removing layers has nothing to do with component classification. We don't have unnecessary interfaces and the restriction of layer encapsulation, but the components still have their scope of work.

The point to highlight in this section is that we should not let Spring's component archetypes and the classic layered architecture fool us about component classification. By following tutorials out there, we tend to believe that a module is made up of 3 components: `Controller`, `Service` and `Repository`. For example, to create `product` module, you would have `ProductController`, `ProdcutService` and `ProductRepository`. With this misbelief, we will eventually add more and more methods to `ProductService` and end up with another kind of anti-pattern, a `God Object`. 

The proper way is to classify your components in more detail. They can be `ProductFactory`, `ProductPromotionManager`, `ProductDiscountCalcualtor`, `ProductOutOfStockNotifier`, etc. that conform to SOLID priciples and make your design easy to explain as much as possible.

# What about Dependency Inversion Priciple?

We have discussed that creating an unnecessary interface and the single implementation, such as `ProductService` and `ProductServiceImpl`, is an anti-pattern. We promote using concrete class unless the abstraction is really needed, which is a good practice according to [YAGNI](https://martinfowler.com/bliki/Yagni.html) priciple. But without the interfaces, how can apply Dependency Inversion Principle (DIP) to our application?

The priciple says

> A: High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces).
> B: Abstractions should not depend on details. Details (classes) should depend on abstractions.

To satisfy the principle, the common practice is to declare all dependencies of a class using interfaces. For example

```
interface ProductService {
    void foo();
    void foo1();
    void foo2();
}
```

```
interface PromotionService {
    void bar();
    void bar1();
    void bar2();
}
```

```
class ProductPromotionManager {

    // All dependencies are interfaces
    public ProductPromotionManagerImpl(ProductService productService, PromotionService promotionService) {
        ...
    }

    public void doSomethingWithProductsAndPromotions() {
        productService.foo();
        promotionService.bar();
        ...
    }
}
```

With the old single implementation anti-pattern, we seem to have no problem with DIP as we already have `ProductService` and `PromotionService` as interfaces. But if we look at it carefully we will find something interesting. Eventhough it satisfies DIP, this pattern seems to violate Interface Segregation Priciple (ISP). Notice that the class `ProductPromotionManager` has only method `doSomethingWithProductsAndPromotions()` which dependes only on the method `foo()` in `ProductService` and metehod `bar()` in `PromotionService`. But the whole interfaces are declared as the dependencies, which make `ProductPromotionManager` depends on the methods it doesn't use, thus violates ISP.

You may argue against this. Because `ProductPromotionManager` does not use `foo1()`, changes in `foo1()` therefore doesn't affect `ProductPromotionManager`. But in pracetice, how do you know that `ProductPromotionManager` does not use `foo1()`? You can draw a dependecy graph to inspect your design but you cannot tell if a developer call `productService.foo1()` somewhere in the class, it is the implementation detail.

To clarify this idea, suppose you are writing a test for `ProductPromotionManager.doSomethingWithProductsAndPromotions()`, the test look like

```
class ProductPromotionManagerTest {

    @Test
    public void testDoSomethingWithProductsAndPromotions() {
        // Arrange
        Mockito.when(productService.foo()).then(...);
        Mockito.when(promotionService.bar()).then(...);

        // Act
        productPromotionManagerImplUnderTest.doSomethingWithProductsAndPromotions();

        // Assert
        ...
    }
}
```

Here we use `Mockito` to mock the methods of the dependecies so that the test does not have to depend on the implementation of the dependencies. Please notice that we mock `productService.foo()` and `promotionService.bar()` before testing `productPromotionManagerImplUnderTest.doSomethingWithProductsAndPromotions()`. The question is, how do we know that we should mock only `foo()` and `bar()` and not `foo1()` and `bar1()` and so on? This means that the developer wrote the test knowing implementation detail, we should not write tests this way.


This example is to show that the old way we implement DIP using sigle implementation anti-pattern already has problem in itself. I would like to propose a better way that

1. Fully satisfies DIP and ISP.
2. Does not force us to use single implementation anti-pattern


The idea is, we don't make `ProductPromotionManager` depends on the other components. Instead, it should decalre the dependencies using plugin pattern.

```
interface ProductFooOpertion {
    void apply();
}
```

```
interface PromotionBarOperation {
    void apply();
}
```

```
class ProductPromotionManager {

    public ProductPromotionManager(ProductFooOpertion productFooOperation, PromotionBarOperation promotionBarOperation) {
        ...
    }

    public void doSomethingWithProductsAndPromotions() {
        productFooOperation.apply();
        promotionBarOperation.apply();
        ...
    }
}
```

In this example, the dependencies are transformed to funcation interfaces `ProductFooOpertion` and `PromotionBarOperation`. These are not components, it is coherent with `ProductPromotionManager`. `ProductService` and `PromotionService` do not know about these interfaces. To declare an instance of `ProductPromotionManager`, we **plugin** the implementation `productSerivce.foo()` and `promotionService.bar()` using lambda methods. As shown in following snippet.

```
class ApplicationConfiguration {

    @Bean
    @Autowire
    public ProductPromotionManager productPromotionManager(ProductService productSerivce, PromotionService promotionService) {
        return new ProductPromotionManager(
            () -> productSerivce.foo(),
            () -> promotionService.bar()
        );
    }
}
```

Now, our code is truely decoupling, conforming to DIP and ISP!

# Conclusion

In this article, we discussed the pain of misplacing layered architecture in enterprise applications. And explore some alternative solution which I hope can help developers or architects improve thire software design toward more OO way.
