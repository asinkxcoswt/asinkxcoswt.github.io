# I18N Domain Model Design With Spring and JPA

Spring has a [built-in approach](https://www.baeldung.com/spring-boot-internationalization) to add I18N nature to applications. The approach only requires that the application provides an implementation of [MessageSource](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/MessageSource.html), which resolves a message for the given locale and message code. This approach is very simple, but limited for only static content such as UI messages. To make our application fully I18N supported. It is inevitable to design our data model with this requirement in mind. Unlike the I18N MessageSource based approach, there is no best practices or standard ways to design I18N data model, we have to rely on our experience but building a dynamic I18N content system has a lot of hidden concerns. Inability to address and handle these concerns early can make the application evolves to a very complex system.

In this article, I will share some of the concerns from my recent project, which uses JPA based data modeling. At the end of the article, I will propose a reusable abstraction that can handle all such concerns, hoping that anyone can apply it in their model to gain I18N support with minimal effort.

# Concerns Checklist For Dynamic I18N Application

Imagine you are asked to build an application that supports dynamic I18N content. The requirement is simple, most of the content can be editable by admins via a back-office website, and they should have multiple languages. Following are concens that are likely to popup from such requirement during development.

### Make it clear if the application should supports adding more languages in future without changing code.

Customers usually like that the system they paid for are as flexible as possible, so without making it clear, it is natural that the customers expect this feature implicitly. For us developers, the answer to this question can turn our application from a small cottage to a skyscrapper in term of complexity. If it is *no*, the data model can be as simple as having some fixed fields for each languages. Following code shows such example.

// i18n_table_design_0.jpg

This design is simple, it maps each field to a database column. It is easy to understand for anyone. The users can search and filter from any provided languages with equally good performance. The UX of the back-office website is also simple, each language is treated as separated fields so the admin must enter the value for every langauges as in the same way we handle other mandatory fields. 

On the other hand, if the answer is *yes*. The design is no longer simple. We may have to separate the model into multiple tables. We will see this in subsequent sections.

### Define the defualt language.

The ability to add more language in future implies that not all languages will have the content specified, which mean we need a strategy to resolve the content for the missing languages. With my naive decision, I used to make it resolves to the first language the record was created with, which is an example of bad design decision a developer can make without talking to the user. Imagine a product in English langauge is misplaced in a list of Thai products in search result because the admin has happened to create the product first in English langauge. Does it make sense to have the admin fix this by removing the product and create a new one in Thai langauge first?

The better approach is to have a default language. A constraint should be made that a record always have content in the default language, otherwise the data is corrupted. With this strategy, the admin is forced to create a record in the default langauge first, the other languages are optional. If a record has some langauge missing, it resolves to the default language, the admin can add the content for the missing language anytime.

### Becareful about search and filter performance.

Suppose you want to sort a list of products in search result by its name, what should the SQL look like? If we use one table per entity record and put `name_th`, `name_en`, `name_jp`, etc. in the same table, we have no problem about sorting and filter, but we cannot add more language dynamically. So we may have an additional table to keep the product data in each language.

// i18n_table_design_1.jpg

Now, to filter and sort the search result by `name` in locale `en`, for example, it cannot be a simple JOIN and WHERE because we have logic to resolve the content of missing languages. Following is an example of SQL that searches products by `:filter_keyword` for language `:current_locale` and sort the result by `name`.

```sql
select p.uuid
, p.price
, coalesce(pd_current.name, pd_defualt.name) name
, coalesce(pd_current.description, pd_defualt.description) description
from products p
join product_details pd_defualt on (pd_defualt.product_uuid = p.uuid and pd_defualt.locale = :default_locale)
left join product_details pd_current on (pd_current.product_uuid = p.uuid and pd_current.locale = :current_locale)
where coalesce(pd_current.name, pd_defualt.name) = :filter_keyword
order by coalesce(pd_current.name, pd_defualt.name)
```

The query is slow because the database system cannot use indexes for this SQL, it has to do a full scan on the join result of `products` and `product_details` for filtering and sorting.

In my recent project, we handle this problem by adding a limitation that multi-language fields are searchable and filterable only in the default langauge. The SQL then can be simplified as follows.

```sql
select p.uuid
, p.price
, coalesce(pd_current.name, pd_defualt.name) name
, coalesce(pd_current.description, pd_defualt.description) description
from products p
join product_details pd_defualt on (pd_defualt.product_uuid = p.uuid and pd_defualt.locale = :default_locale)
left join product_details pd_current on (pd_current.product_uuid = p.uuid and pd_current.locale = :current_locale)
where pd_defualt.name = :filter_keyword
order pd_defualt.name
```

This improves the query performance given that we have an index on the column `name`. In addition, we can tweak it a bit by moving the data of the default language to the main `products` table, leaving only additional languages in the table `product_details`.

// i18n_table_design_2.jpg

With this design, the sortable and filterable fields are grouped in one place, in the main table. We can establish a standard for our application that we allow search and sort only on the fields in the main table, we don't have to make a document to specified which fields are sortable and filterable. Also, the query performance is better compared to the previous design because the filtering is done on `products` table which has less records than `product_details` and we don't have to join `product_details` two times.

```sql
select p.uuid
, p.price
, coalesce(pd_current.name, p.name) name
, coalesce(pd_current.description, p.description) description
from products p
left join product_details pd on (pd.product_uuid = p.uuid and pd.locale = :current_locale)
where p.name = :filter_keyword
order p.name
```

We can successfully solve the performance problem by adding this limitation because this SQL based search and filter feature is used only in back-office website, by admins, in which we enforce that the list pages (e.g. product list, user list, post list, etc.) can show content only in the default language. For the front-end website, we can use search engine such as [Solr](http://lucene.apache.org/solr) or [Elasticsearch](https://www.elastic.co).

### Determine content negotiation machanics.

Some websites have a kind of *change-language* button somewhere for the user that want to see the content in other languages. While some websites have no button, but resolve the content's language based on the actual location of the user by using the standard HTTP header `Accept-Language`. It's best that we always make our APIs support both cases.

In case the client application does not have the *change-language* button, or it is the first time the user accesses the website, the APIs should resolve the language based on the user's location using the header `Accept-Language`. Following are some examples.

```
Request -> GET /products/594d9cd0-588a-11e9-9244-d66b27764bfe
Header -> "Accept-Language: en"
Response ->
{
    "uuid": "594d9cd0-588a-11e9-9244-d66b27764bfe"
    "name": "Samsung Galaxy",
    "description": "A mobile phone",
    "price": 20000
}
```

```
Request -> GET /products/594d9cd0-588a-11e9-9244-d66b27764bfe
Header -> "Accept-Language: th"
Response ->
{
    "uuid": "594d9cd0-588a-11e9-9244-d66b27764bfe"
    "name": "ซัมซุง กาแล็กซี",
    "description": "โทรศัพท์มือถือ",
    "price": 20000
}
```

The value of header `Accept-Language` is set by the browser. As stated in [the standard](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language), the client application should not alter the value of this header.

When the user chooses an option from the *change-language* button, the client application then makes a request to the API by placing the specified language as a request parameter (or using cookies in stateful environment).

```
Request -> GET /products/594d9cd0-588a-11e9-9244-d66b27764bfe?language=en
Response ->
{
    "uuid": "594d9cd0-588a-11e9-9244-d66b27764bfe"
    "name": "Samsung Galaxy",
    "description": "A mobile phone",
    "price": 20000
}
```

In either case, the API is responsible to resolve the content for the given language. Please notice, in the examples above, that the data model hides how the back-end handles I18N. The client does not know about `name_th`, `name_en` and the like. Also It does not concern if the value of the field `name` is from table `products` or `product_detialas` or whether the content is actually of the given language or not. All to logic is on the API.

However, an open question is left to the reader to determine if the language resolving logic should be on the API. In some situations, it might be possible that the API provides all possible language to the client and let the client resolves the content to display.

```
Request -> GET /products/594d9cd0-588a-11e9-9244-d66b27764bfe
Response ->
{
    "uuid": "594d9cd0-588a-11e9-9244-d66b27764bfe"
    "name": "Samsung Galaxy",
    "description": "A mobile phone",
    "price": 20000,
    "additionalLocalizedData": {
        "th" : {
            "name": "ซัมซุง กาแล็กซี",
            "description: "โทรศัพท์มือถือ"
        },
        "jp" : {
            "name": "サムスン ギャラクシー",
            "description: "携帯電話"
        },
    }
}
```

### Dynamic I18N Data modeling is not easy with JPA

Dynamic I18N requirement can be a reason to consider against using ORM based approach such as JPA in our application. Applications will be much more simpler without I18N concerns. In a non-JPA application, we can hide the complexity of language resolving inside the database mapping layer, and leave the application model as simple as possible. Let's see some examples to clarify this idea.

#### Example of Data Model Without I18N Concerns

```java
interface Product {
    UUID getUuid();
    String getName(); // multi-language
    String getDescription(); // multi-language
    String getPrice();
}
```

```java
interface ProductRepository {
    Product findByUuidAndLocale(UUID productUuid, Locale locale);
}
```

```sql
select p.uuid
, p.price
, coalesce(pd_current.name, p.name) name
, coalesce(pd_current.description, p.description) description
from products p
left join product_details pd on (pd.product_uuid = p.uuid and pd.locale = :current_locale)
where p.uuid = :product_uuid;
```

In this example, Our `Product` model have to support multile languages for fields `name` and `description`, but we design it as if it supports only one language. The responsibility to resolve the language is delegated to `ProductRepository`. We omit the full implementation of the repository for brevity and show only the SQL that can map the complex database tables to our simple `Product` model. Later in our application code, we can conveniently use the `Product` without having to concern about its language.

#### Example of I18N Data Model With JPA

JPA can help hiding the complexity of the database mapping in general, but it has less flexibility for us to customize the mapping logic, so it is not easy to design the model in JPA as simple as in the previous example. With the minimal attempt, our JPA model for `Product` may look like this, which is much more complex.

```java
@Entity
class Product {
    @Id
    private UUID uuid = UUID.randomUUID();

    @Column
    private BigDecimal price;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @MapKey(name = "locale")
    private Map<Locale, ProductDetail> detailMap = new HashMap<>();

    public ProductDetail getDetailForLocale(Locale locale) {
        ProductDetail detail = detailMap.get(locale);
        if (detail != null) {
            return detail;
        }
        
        detail = detailMap.get(ApplicationSetting.DEFAULT_LOCALE);

        if (detail != null) {
            return detail;
        }

        throw new SomeRuntimeException("Data corrupt, not found default locale for product");
    }

    public void addDetail(ProductDetail detail) {
        detailMap.put(detail.getLocale, detail);
        if (detailMap.get(ApplicationSetting.DEFAULT_LOCALE) == null) {
            detailMap.put(ApplicationSetting.DEFAULT_LOCALE, detail);
        }
    }

    ...
}
```

```java
@Entity
class ProductDetail {
    @Id
    private UUID uuid = UUID.randomUUID();

    @Column
    private Locale locale;

    @Column
    private String name;

    @Column
    private String description;
}
```

In this example, we have to create a model for the main `Product` and another model `ProductDetail` to hold a product data for a language (locale). Many logic to handling the detail per language goes in `Product` class. The object graph of your software design will have another entity `ProductDetail` that has no real meaning for the business domain.

# Hiding I18N Complexity With Domain Behavior Framework

In the previous section, we talk about various concerns in designing dynamic I18N content system. In this section, we will see how to design a domain model that respects all such concerns.

- We should be able to add more arbitrary languages without changing code.
- The system should resolve content for missing languages to the default language.
- To handle the performance issue in search and filter, we use the master-detail pattern for database table design, where the master table has all the same field set as in the detail table, to keep the data for the default language and to enfore that the content always exists in the default lanuguage. And we allow search and filter only on fields in the master table.
- The language resolving logic should be hidden from the client of the domain class. The interface of the domain class will be designed as if no internalization is concerned, to preserve its meaning to the actual business domain.
- We will leverage the convenience of JPA in database mapping, while overcome its downside in modeling flexibility by the help of [Domain Behavior Framework](https://github.com/asinkxcoswt/domain-collections).

> [Domain Behavior Framework](https://github.com/asinkxcoswt/domain-collections) is my attempt to add Spring bean power to domain classes by mean of Mixin Pattern. It is to overcome the limitation that Spring DI only works with registered beans while the domain classes usually are POJOs instantiated by the underlying JPA framework. The domain classes can reclaim their behaviors and be free from being anemic models, while still conform to the single responsibility principle as each behavior is implemented in its own mixin class.

### The final domain class

First, let's see what the final model looks like.

```java
@Entity
@Table(name = "product_details")
class ProductDetail implements I18NDetail {
    @Id
    private UUID uuid; // with getters, setters

    @Column
    private String name; // with getters, setters

    @Column
    private String description; // with getters, setters

    @Column
    private Locale locale; // with getters, setters
}

```

```java
@Entity
@Table(name = "products")
class Product extends ProductDetail implements DomainBehavior, I18NSupport<ProductDetail>, SettingSupport {

    @Transient
    @DomainBehaviorProxy
    private I18NFooEntity self;

    @Column
    private BigDecimal price; // with getters, setters

    @OneToMany
    @MapKey(...)
    private Map<Locale, ProductDetail> detailMap = new HashMap<>();

    @Override
    public Locale getLocale() {
        Setting setting = self.getSettingValue(ApplicationSettingKeys.DEFAULT_LOCALE);
        return Locale.forLanguageTag(setting.getValue(String.class));
    }

    @Override
    public void setLocale(Locale locale) {
        throw new UnsupportedOperationException();
    }

    ...
}
```

We have two JPA entities, `Product` and `ProductDetail` corresponding to the master table `products` and the detail table `product_details` respectively. `ProductDetail` has only multi-language fields. The `Product` extends `ProductDetail` so it has the fields to keep the data for the default language. The method `getLocale()` and `setLocale()` are overriden so that the locale for the master entity is always the default language. The `SettingSupport` is another convenient mixin that allows the entity class to retrieve setting values form database, but we will not go into detail about it here. The `Product` entity implements `DomainBehavior`. This is the marker interface to make this entity eligible to be wrapped inside a Spring CGlib proxy that enable the Domain Behavior nature.

The mixin interface `I18NSupport` is the key in our modeling. It adds I18N nature to the implemented entity. The interface is declared as follows.

```java
public interface I18NSupport<D extends I18NDetail> {

    @JsonIgnore
    Optional<D> getI18NDetailForLocale(Locale locale);

    @JsonIgnore
    D getDefaultI18NDetail();

    @JsonIgnore
    Class<D> getI18NDetailClass();

    @JsonIgnore
    Locale getCurrentLocale();

    @JsonIgnore
    D initializeI18NDetailForLocale(Locale locale);
}
```

```java
public interface I18NDetail {
    Locale getLocale();
    void setLocale(Locale locale);
}
```

There are 5 methods the entity class has to implement for `I18NSupport`, following code shows how to do it in `Product` class, which should be self-explanatory.

```java
@Entity
@Table(name = "products")
class Product extends ProductDetail implements DomainBehavior, I18NSupport<ProductDetail>, SettingSupport {
    @Column
    private BigDecimal price; // with getters, setters

    @OneToMany
    @MapKey(name = "locale")
    private Map<Locale, ProductDetail> detailMap = new HashMap<>();

    ...

    @Override
    public Optional<I18NFooDetail> getI18NDetailForLocale(Locale locale) {
        return Optional.ofNullable(fooDetailMap.get(locale));
    }

    @Override
    public I18NFooDetail getDefaultI18NDetail() {
        return this;
    }

    @Override
    public Class<I18NFooDetail> getI18NDetailClass() {
        return I18NFooDetail.class;
    }

    @Override
    public Locale getCurrentLocale() {
        return LocaleContextHolder.getLocale();
    }

    @Override
    public I18NFooDetail initializeI18NDetailForLocale(Locale locale) {
        I18NFooDetail detail = new I18NFooDetail();
        detail.setLocale(locale);
        this.detailMap.put(locale, detail);
        return detail;
    }
}
```

Following is the minimal configuration that brings about the magic.

```java
@EnableJpaRepositories(repositoryFactoryBeanClass = DomainBehaviorSupportJpaRepositoryFactoryBean.class)
@SpringBootApplication
public class I18NSupportTestConfiguration {
    @Bean
    public SpringApplicationContextHolder springApplicationContextHolder() {
        return new SpringApplicationContextHolder();
    }

    @Bean
    public SettingBehavior settingBehavior(ApplicationSettingRepository applicationSettingRepository) {
        return new SettingBehavior(applicationSettingRepository);
    }

    @Bean I18NAdvice i18NAdvice() {
        return new I18NAdvice();
    }

    @Bean
    public DomainBehaviorManager domainBehaviorManager() {
        return new DomainBehaviorManager();
    }

    @Autowired
    public void configure(DomainBehaviorManager domainBehaviorManager,
                          SettingBehavior settingBehavior,
                          I18NAdvice i18NAdvice) {

        domainBehaviorManager.registerBehavior(I18NSupport.class, i18NAdvice);
        domainBehaviorManager.registerBehavior(SettingSupport.class, settingBehavior);
        domainBehaviorManager.afterPropertiesSet();
    }
}
```

Finally, we can use the domain class without having to concern about its langauge. The `I18NAdvice` do all the work for us. See some example usages from following test.

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {I18NSupportTestConfiguration.class})
public class I18NSupportTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ApplicationSettingRepository applicationSettingRepository;

    @Test
    public void test() throws NoSuchFieldException, IllegalAccessException {
        // prepare setting DEFAULT_LOCALE = en
        applicationSettingRepository.save(new ApplicationSetting(ApplicationSettingKeys.DEFAULT_LOCALE, "en"));

        // prepare a product for test
        Product product = productRepository.create("Samsung Galaxy", "A mobile phone", BigDecimal.valueOf(20000));
        productRepository.save(product);

        // check that the setting default locale works
        Assertions.assertEquals(Locale.forLanguageTag("en"), product.getDefaultI18NDetail().getLocale());

        // now let's change the language to th
        LocaleContextHolder.setLocale(Locale.forLanguageTag("th"));

        // check that the name is resolved to the default language because we have no name in th
        Assertions.assertEquals("Samsung Galaxy", product.getName());

        // let's set the name in language th
        product.setName("ซัมซุง กาแล็กซี");
        productRepository.flush();

        // now the name is resolved to Thai language
        Assertions.assertEquals("ซัมซุง กาแล็กซี", product.getName());

        // the ProductDetail where locale = th was created
        Assertions.assertTrue(product.getI18NDetailForLocale(Locale.forLanguageTag("th")).isPresent());

        // what about changing the language to something else
        LocaleContextHolder.setLocale(Locale.forLanguageTag("jp"));

        // the name is still resolved to the default language
        Assertions.assertEquals("Samsung Galaxy", product.getName());

        // let's set the name in language jp
        product.setName("サムスン ギャラクシー");
        productRepository.flush();

        // now the name is resolved to Japanese language
        Assertions.assertEquals("サムスン ギャラクシー", product.getName());

        // the ProductDetail where locale = jp was created
        Assertions.assertTrue(product.getI18NDetailForLocale(Locale.forLanguageTag("jp")).isPresent());

        // the data for Thai language is still there
        LocaleContextHolder.setLocale(Locale.forLanguageTag("th"));
        Assertions.assertEquals("ซัมซุง กาแล็กซี", product.getName());

        // what if the current language is equal to the default language
        LocaleContextHolder.setLocale(Locale.forLanguageTag("en"));

        // of course, the name should be in the default language
        Assertions.assertEquals("Samsung Galaxy", product.getName());

        // but if we set the name to something else,
        product.setName("Iphone X");

        // there should be no record in ProductDetail where locale = en, the value should be set directly to Product
        Assertions.assertFalse(product.getI18NDetailForLocale(Locale.forLanguageTag("en")).isPresent());

        // the default name should be changed to the new value
        LocaleContextHolder.setLocale(Locale.forLanguageTag("ch"));
        Assertions.assertEquals("Iphone X", product.getName());
    }
}
```

# Conclusion

Designing dynamic I18N system from scratch is not an easy job. This article summarized the result from trials and errors and provide an abstraction that anyone can extend. Please see [the readme page of this repository](https://github.com/asinkxcoswt/domain-collections) to get more idea about what is the Domain Behavior Framework. The full working source code for I18N support can be found in [this package](https://github.com/asinkxcoswt/domain-collections/tree/master/src/test/java/com/asinkxcoswt/domain/behavior/i18n_support).