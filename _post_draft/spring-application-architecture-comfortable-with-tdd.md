TDD is like a religion. Its doctrine is simple while can be interpreted subjectively. You achieve its goal by practicing rather than following processes. While some followers success, many of them [give up](https://dhh.dk/2014/tdd-is-dead-long-live-testing.html). Some experts have an explanation about [why TDD is not working for someone](https://dev.to/arminaskatilius/why-tdd-is-not-working-for-you-2hbd), a common reason is they are too strict to TDD, tempted to believe that it is the only right way, or it alone is enough, to develop a software. In fact, TDD is just a little tool. You are free to decide to use it when you are comfortable with it and choose other solutions elsewhere.

Because TDD itself does not define a strict process to follow. A TDD beginner like me is trying to pursue what the others are doing. It turns out to be a bigger issue than I thought. It involves setting up a solid development process, automated pipeline, and people mindsets. It seems to be of an organization scale changes, and too big to be able to focus on which problem actually we are trying to solve with such changes.

As a little gear in a big organization, I am trying to take a baby step exploring some little points where TDD can solve my existing problems. It should be something that we can start doing today or tomorrow without having to turn the organization around.

# TDD As A Conversation Medium

As a developer struggling to learn about the correct way to do software development, I have a funny point of view to the process we have done in my recent project. 

I will try not to talk about what development process we have used or whether we have done it right as it is a very big topic. However, I believe that most of us have the following steps in mind when we want to build something.

![image](uploads/022353c3229ef6678793987a2205254e/image.png)

The funny part is that these simple steps are just a variant of telephone game where people transfer their understanding of the source image from one person to another to challenge that the understanding will be transferred to the final person correctly.

![image](uploads/34aacdecd51e85b4702415fbc21088c9/image.png)

The problem that makes this game challenged is that the players are forced to do one-way communication using an unreliable medium. 

In software development, we are tempted to optimize our work to achieve a perfect telephone game. We dream of the ideal where the work flows in one way seamlessly. As a result, development becomes like this.

![image](uploads/ff9ec02e5cf1b7a570e378e08270f34c/image.png)
    
We treat imperfect specification as a root cause of ETA misestimation. We think the project is delayed because developers have to spend time discussing things instead of writing code. We see it as a problem that should be eliminated. Unfortunately, we cannot eliminate it however we have tried. Maybe we should admit that it is nature rather than a problem so that we can prepare for it.

Software development is a learning process toward understanding the business domain. The requirement might ask us to create a mystical big elephant no one have ever seen before. The business team learns about the elephant by exploring user needs, while developers use the code and logic to learn the technical aspect of the elephant. Writing software, in this case, cannot be a one-way communication where someone defines a perfect specification and let the other write the code. Instead, it needs CONVERSATIONS among the team members. The specification is not a blueprint to write software, it is just one of the possible mediums to convey the image of the elephant.

I do not mean we should discard all the standard software specification documents. At least, we need something to communicate with stakeholders about what to expect for the software. Instead, I think we should improve COMMUNICATION inside the development team. The specification is just one of possible medium for communication. and it alone is not enough as it is not a good medium for CONVERSATION.

After exploring how people do TDD, I think the test code fits very well to be the place where development team converses. It should be made a proprietary of the development team rather than just the developers and testers. Example usages are as follows.

- BA, SA or even PM can initiate a test suite or add more test cases to convey their expectations for a requirement. 
- Developers then can develop production code to meet the expectations as well as adding more test cases as they find more aspect of the requirement during design or writing the code. 
- Front-end developers can add test cases for the back-end guys to express their expectation of the APIs, and continue working on that assumption without having to wait for the back-end to complete the test.
- When the developers find some cases that need someone to verify, they can use a test case as a question to the team, asking someone to fill in the expectation using the test language.

And the following are the benefits.

- The language of the test usually is more explicit than human language written in a text document or spreadsheet.
- SA and BA don't have to add too many details to the specification document to communicate with stakeholders, making the document more readable for everyone. The technical details are put in test suites to communicate with developers.
- Software expectation is not driven in one way by analysts to developers. Instead, everyone in the development team helps drive the software together.
- The test suite act as a definition of done for a story. It can be used to track the progress of the development.
- It cannot be outdated as it is put into the same source code repository as the production code, leverage the benefit of the version control tool.
- Remote work is made easier as the communication is explicit and shared remotely in the repository.
- By putting all the expected cases in a test suite before writing code, everyone in the development team sees together which cases are committed to be delivered on the due date and which are added in later. We can prioritize the work better by postponing some cases. The pending cases are still there, we don't have to fear that we will forget it. And we don't have to waste time re-discuss the issue as all the details and expectation are expressed in the test code.

We will see an example of this in the last section.

# TDD As A Pattern Enforcer

Suppose you are building an apartment of 10 rooms. At the first room, you spend some time to learn about how things should be placed in the room. The other rooms, then, are to be built using the same pattern, which mean you should work faster on them because you have already known about the pattern of things in the room. For example, you know the amount of concrete to build a bathroom or how many tiles are required for the floor. To put it in software terms, you have established the pattern for each component in a room and reuse them in other rooms.

Features in software are similar to rooms in an apartment, they share more or less the same kind of components. In some case, developers can duplicate the entire structure of an existing feature and then modify it to make up a new feature. Though, an exception to this is when we have a requirement for a new kind of feature. In this case, we will have to take time inventing the new pattern. 

A component archetype is a component that holds patterns. And it is critically important in software development. They are similar to how we summarize complex concepts into one word, and then use the word to build up the more complex concept. Imagine you are asking a developer to build a mystical smartphone, it will be much easier if the developer already knows what a smartphone is. Otherwise, you have to waste time explaining the definition of a smartphone.

An example component archetype in Spring RESTful APIs development is a `RestController`. It is a component where we define the URL mapping of an API as well as its interface. To develop an application, we often have to add more specific requirements to this component archetype to establish standard throughout our application. Following are some examples of such requirements.

- It should handle an authorization error appropriately.
- It should handle an internal server error appropriately.
- It should handle a not-found error appropriately.
- It should handle a happy case appropriately.
- It should handle an input-validation error appropriately.
- It should not use HTTP method other than GET, PUT, POST, DELETE
- Every GET method should support cache control in the response header
- It should have the URL that conforms RESTful naming convention

Now that we have the pattern for an archetype, the next thing to do is to find a good name. Let's call it a `StandardRestController`. Next time when a developer is asked to create a `StandardRestController` that can do X, Y, and Z, he should know immediately that his work does not only include X, Y and Z, but also all the implicit requirements of the archetype.

At this stage, we have a good component archetype. But we have to have faith in fellow developers to sustain it. Using faith is not effective as we will eventually lose it. So the idea is we should find a way to enforce it.

TDD together with source code generation technique can be used to enforce the archetype's implicit requirements. I have created [an annotation processing tool](https://github.com/asinkxcoswt/archetype) that can do this work. To use it, include the following dependency into your project's `pom.xml`.

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>com.github.asinkxcoswt</groupId>
        <artifactId>archetype</artifactId>
        <version>V1.0.0-beta</version>
    </dependency>
</dependencies>
```

Next, create the controller class with the annotation.

```java
@Archetype(template = "archetype/standard-rest-controller.mustache")
@Controller
pulbic class ProductQueryApi {
    @GetMapping("/products/{productUuid}")
    public ResponseEntity findProduct(@PathVariable String productUuid) {
        return ResponseEntity.ok().build();
    }
}
```

The corresponding test class will be created in the same package under the `test` directory, which looks like this.

```java
@SpringBootTest
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class ProductQueryApiTest {

    private MockMvc mockMvc;

    @BeforeEach
    public void setUp(WebApplicationContext context, RestDocumentationContextProvider restDocContextProvider) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(context)
                .apply(documentationConfiguration(restDocContextProvider))
                .build();
    }

    @Test
    public void testFindProduct_ok() throws Exception {
        this.mockMvc.perform(get("/products/{productUuid}", "4055b7b8-ebe4-4f52-8de0-c99a84862857").header("Authorization", "a-valid-token"))
                .andExpect(status().isOk())
                .andDo(
                        document("find_product_ok",
                                responseFields(fieldWithPath("name").type("string").description("The name of the product"))));
    }

    @Test
    public void testFindProduct_withInvalidAccessToken() throws Exception {
        this.mockMvc.perform(get("/products/{productUuid}", "4055b7b8-ebe4-4f52-8de0-c99a84862857").header("Authorization", "an-invalid-token"))
                .andExpect(status().is(HttpStatus.UNAUTHORIZED.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("unauthorized"))
                .andDo(
                        document("find_product_unauthorized"));
    }

    @Test
    public void testFindProduct_internalError() throws Exception {
        this.mockMvc.perform(get("/products/{productUuid}", "7b8664ee-6451-11e9-a923-1681be663d3e").header("Authorization", "a-valid-token"))
                .andExpect(status().is(HttpStatus.INTERNAL_SERVER_ERROR.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("server_error"))
                .andDo(
                        document("find_product_internal_error"));
    }

    @Test
    public void testFindProduct_notFound() throws Exception {
        this.mockMvc.perform(get("/products/{productUuid}", "7b8660de-6451-11e9-a923-1681be663d3e").header("Authorization", "a-valid-token"))
                .andExpect(status().is(HttpStatus.NOT_FOUND.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("not_found"))
                .andDo(
                        document("find_product_not_found"));
    }

    @Test
    public void testFindProduct_badRequest() throws Exception {
        this.mockMvc.perform(get("/products/{productUuid}", "not-a-uuid").header("Authorization", "a-valid-token"))
                .andExpect(status().is(HttpStatus.BAD_REQUEST.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("invalid_uuid"))
                .andDo(
                        document("find_product_bad_request"));
    }
}
```

Please notice that the test cases do not have to be completed as the requirements are specific to each API. Instead, we just have to generate the test cases that fail to enforce the developer to look into the test and define the appropriate expectations for each case. If the developer cannot decide the expectation for some case, he can notify the team and have someone to fill the expectation, as suggested in the previous section.

The side benefit of this approach is that our software will be enforced to have a unit test counterpart for every archetyped component. It encourages the developer to do TDD. He can see how the test should be written from the generated cases and mimic them to create their own additional test cases.

Conceptually, we can have many component archetypes as we like. But I recommend that we should define an archetype for every component that is at the edge of the architecture, such as the controller. Testing the application at its edge can be done up-front before the detailed design is done. It is the place where we have discussed in the previous section that the development team should take time to nurture.

# Spring Rest Docs

In the previous section, we show an example of how to use TDD for a RESTful controller. We did not mention, however, that the syntax we use in the test class is from Spring Rest Docs. It is a test framework that helps us generate the API documentation from the test cases. Please see the official guide for full detail on how to do it. The following picture is the documentation generated from our previous example. We mention this feature to encourage the reader that your effort on writing test, not only it helps to improve the team communication, but also your every effort are put in the document!

![image](uploads/c77f0062c51b227a7fc63fa024ecce09/image.png)

# In Action

We have discussed the idea to use testing as a conversation tool and how the archetype enforcer can help us bootstrap the test cases for API development. Let's see the full development process in action.

Suppose the requirement is to create an API for adding a product to the authenticated user's shopping cart. Marry is a SA, she has some initial ideas about what the API should handle. So she opens the code editor and creates this class.

```java
@RestController
@Archetype(template = "archetype/standard-rest-controller.mustache")
public class ShoppingCartApi {
    @PostMapping("/shopping-cart/items")
    public ResponseEntity addItemToCart() { return ResponseEntity.status(HttpStatus.NOT_IMPLEMENTED).build(); }
}
```

After compiled, the test class `ShoppingCartApiTest` will be automatically generated with some implicit test cases.

```java
@SpringBootTest
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class ShoppingCartApiTest {

    @Test
    public void testAddItemToCart_ok() throws Exception { ... }

    @Test
    public void testAddItemToCart_unauthorized() throws Exception { ... }

    @Test
    public void testAddItemToCart_bad_request() throws Exception { ... }

    @Test
    public void testAddItemToCart_internal_error() throws Exception { ... }
}
```

Mary has, in her mind, the input and output of the API and some error cases that should be handled. So she adds them into the test class.

```java
@SpringBootTest
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class ShoppingCartApiTest {

    @Test
    public void testAddItemToCart_ok() throws Exception {
        this.mockMvc.perform(post("/shopping-cart/items")
                .header("Authorization", "a-valid-token")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productUuid\": \"7b8660de-6451-11e9-a923-1681be663d3e\", \"quantity\": 10}"))

                .andExpect(status().isOk())
                .andDo(
                        document("add_to_cart_ok",
                                requestFields(
                                        fieldWithPath("productUuid").type("UUID").description("The uuid of the product."),
                                        fieldWithPath("quantity").type("Integer").description("The amount to buy."))));
    }

    @Test
    public void testAddItemToCart_quantityExceed() throws Exception {
        this.mockMvc.perform(post("/shopping-cart/items")
                .header("Authorization", "a-valid-token")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productUuid\": \"7b8660de-6451-11e9-a923-1681be663d3e\", \"quantity\": 11}"))

                .andExpect(status().is(HttpStatus.BAD_REQUEST.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("shopping_cart_item_quantity_exceed"))
                .andExpect(MockMvcResultMatchers.jsonPath("message").value("You can buy product up to 10 piece at a time."))
                .andDo(
                        document("add_to_cart_quantity_exceed",
                                responseFields(
                                        fieldWithPath("errorCode").type("String").description("The error code."),
                                        fieldWithPath("message").type("Integer").description("The error message"))));
    }
}
```

John is a developer working remotely. After receiving a message from Mary that the API is ready to be implemented, he pulls the code from the repository and runs the test to see all the expectations.

![image](uploads/4e0a1b5a9f2881bfa02c75f0014fcd7d/image.png)

John then starts writing the production code, which includes all detailed classes behind the scene such as `ShoppingCart`, `ShppingCartRepository`, `ShoppingCartItemFactory`, etc. How John designs these classes are out of Mary's concerns. At the end of the day, John can make the first test case pass. He commits his code the repository. The CI/CD pipeline runs and shows the progress of the add-to-cart story using the test results in `ShoppingCartApiTest`.

At the next day, John continues his work making all the tests pass. But he finds that a case must be handled that Mary did not specify. So he adds a test case to explain the scenario and submit the code to the repository.

```java
    @Test
    public void testAddItemToCart_productNotAvailableToBuy() throws Exception {
        this.mockMvc.perform(post("/shopping-cart/items")
                .header("Authorization", "a-valid-token")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productUuid\": \"4055b7b8-ebe4-4f52-8de0-c99a84862857\", \"quantity\": 10}"))

                .andExpect(status().is(HttpStatus.BAD_REQUEST.value()))
add_to_cart_product_not_available_to_buy                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("shopping_cart_item_product_not_available_to_buy"))
                .andExpect(MockMvcResultMatchers.jsonPath("message").value("???"))
                .andDo(
                        document("add_to_cart_product_not_available_to_buy",
                                responseFields(
                                        fieldWithPath("errorCode").type("String").description("The error code."),
                                        fieldWithPath("message").type("Integer").description("The error message"))));
    }
```

After notified by John, Mary pull the source code, fill in her expectation and push the code back to the repository.

```java
.andExpect(MockMvcResultMatchers.jsonPath("message").value("Sorry, the product is not available."))
```

John then implements the production code to meet Mary's expectation and finish the job. The development team then has new documentation for the shopping cart API.

![image](uploads/a757e84ee9bcb7de3b40f67259d9a421/image.png)

Bob is a front-end developer, also working remotely. He is in charge of using John's API to make the UI. He reads the documentation and sees how the API works and what are possible error cases that should be handled. During the implementation, Bob finds that the API should have a new error code for a specific error case, so he adds a test and continues developing the UI assuming the error code from the API will be according to his expectation written in the test. However, he forgets to notify John about this.

```java
    @Test
    public void testAddItemToCart_productHasNotEnoughQuantity() throws Exception {
        this.mockMvc.perform(post("/shopping-cart/items")
                .header("Authorization", "a-valid-token")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productUuid\": \"7b8664ee-6451-11e9-a923-1681be663d3e\", \"quantity\": 10}"))

                .andExpect(status().is(HttpStatus.BAD_REQUEST.value()))
                .andExpect(MockMvcResultMatchers.jsonPath("errorCode").value("shopping_cart_item_product_has_not_enough_quantity"))
                .andExpect(MockMvcResultMatchers.jsonPath("message").value("Sorry, the product has not enough quantity left."))
                .andDo(
                        document("add_to_cart_product_has_not_enough_quantity",
                                responseFields(
                                        fieldWithPath("errorCode").type("String").description("The error code."),
                                        fieldWithPath("message").type("Integer").description("The error message"))));
    }
```

It's the last day before the due date, sadly, John and Bob take leave. Mary sees the progress of the add-to-cart story and finds that it is not completed due to the test that fails because the API has not handled the new error case yet.

So she asks Sarah, another back-end developer, to fix the API. Sarah takes some time learning about the API by reading the documentation and the test cases, after which she easily discovers where to add the code. She does not have to fear that her change will introduce some bugs as the API has the tests as the safety net. Sarah submits the code to the repository and the story ends happily.
