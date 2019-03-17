```
@Entity
class Product {

    private BigDecimal price;

    public BigDecimal getPriceIncludeVat() {
        BigDecimal vatRate = ... // How to get the vat rate configured in database?
        return price.multiply(vat).divided(BigDecimal.valueOf(100));
    }
}
```

```
interface ApplicationSettingSupport {
    <T> T getApplicationSettingValue(String settingKey, Class<T> bindingClass);
}

@Entity
class Product implements ApplicationSettingSupport {

    private BigDecimal price;

    public BigDecimal getPriceIncludeVat() {
        BigDecimal vatRate = getApplicationSettingValue("VAT_RATE", BigDecimal.class);
        return price.multiply(vat).divided(BigDecimal.valueOf(100));
    }
}
```

```
class ApplicationSettingBehavior implements ApplicationSettingSupport {

    private JsonMapper jsonMapper;
    private ApplicationSettingRepository applicationSettingRepository;
    public ApplicationSettingBehavior(JsonMapper jsonMapper, ApplicationSettingRepository applicationSettingRepository) {
        this.jsonMapper = jsonMapper;
        this.applicationSettingRepository = applicationSettingRepository;
    }

    @Override
    public <T> T getApplicationSettingValue(String settingKey, Class<T> bindingClass) {
        ApplicationSetting setting = applicationSettingRepository.findBySettingKey(settingKey);
        return jsonMapper.read(setting.getValue(), bindingClass);
    }
}
```


------------------------
DomainBehaviorManager
------------------------
registerBehavior(interfaceClass, implementationObject)
afterPropertiesSet()
wrapBehaviors(T domainObject): T
supports(method): Boolean
------------------------


```
class ProductFactory {

    private DomainBehaviorManager domainBehaviorManager;
    public ProductFactory(DomainBehaviorManager domainBehaviorManager) {
        this.domainBehaviorManager = domainBehaviorManager;
    }

    public Product createNewProduct() {
        return domainBehaviorManager.wrapBehaviors(new Product());
    }
}
```

```
@Configuration
class DomainConfiguration {

    @Bean 
    public ApplicationSettingBehavior applicationSettingBehavior(JsonMapper jsonMapper, ApplicationSettingRepository applicationSettingRepository) {
        return new ApplicationSettingBehavior(jsonMapper, applicationSettingRepository);
    }

    @Bean
    public DomainBehaviorManager domainBehaviorManager(ApplicationSettingBehavior applicationSettingBehavior) {
        DomainBehaviorManager m = new DomainBehaviorManager();
        m.registerBehavior(ApplicationSettingSupport.class, applicationSettingBehavior);
        m.afterPropertiesSet();
        return m;
    }
}
```

```
public class DomainBehaviorManager {
    private Map<Class<?>, Object> behaviorMap = new HashMap<>();
    private Mixin mixin;
    public <INTERFACE, IMPL extends INTERFACE> void registerBehavior(Class<INTERFACE> interfaceClass, IMPL impl) {
        this.behaviorMap.put(interfaceClass, impl);
    }

    public void afterPropertiesSet() {
        List<Class<?>> interfaceClassList = new ArrayList<>();
        List<Object> implList = new ArrayList<>();
        for (Map.Entry<Class<?>, Object> entry : this.behaviorMap.entrySet()) {
            interfaceClassList.add(entry.getKey());
            implList.add(entry.getValue());
        }
        this.mixin = Mixin.create(interfaceClassList.toArray(new Class[0]), implList.toArray());
    }

    public boolean supports(Method method) {
        return this.behaviorMap.containsKey(method.getDeclaringClass()) &&
                method.isAnnotationPresent(DomainBehaviorTarget.class);
    }

    public <T> T wrapBehaviors(T domainObject) {
        if (this.mixin == null) {
            throw new IllegalStateException("Mixins are not finalized, forgot to call DomainBehaviorManager::afterPropertiesSet ?");
        }

        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(domainObject);
        proxyFactory.addAdvice((MethodInterceptor) methodInvocation -> {
            Method method = methodInvocation.getMethod();

            if (supports(method)) {
                return method.invoke(mixin, methodInvocation.getArguments());
            } else {
                return methodInvocation.proceed();
            }
        });

        return (T) proxyFactory.getProxy();
    }
}
```

```
@Retention(RetentionPolicy.RUNTIME)
public @interface DomainBehaviorTarget {
}
```


```
interface ApplicationSettingSupport {
    @DomainBehaviorTarget
    <T> T getApplicationSettingValue(String settingKey, Class<T> bindingClass);
}
```

```
@ExtendedWith(SpringExtension.class)
class ProductWithApplicationSettingSupportTest {
    @Test
    public void shouldReturnVatIncludedPriceAccordingToVatRateSetting() {
        Product product = productFactory.createNewProduct();
        applicationSettingRepository.updateSettingValueBySettingKey("VAT_RATE", 3);

        BigDecimal price1 = product.getPriceIncludedVat();
        Assertions.assertEquals(0, price1);

        applicationSettingRepository.updateSettingValueBySettingKey("VAT_RATE", 7);

        BigDecimal price2 = product.getPriceIncludedVat();
        Assertions.assertEquals(0, price2);
    }
}
```

```
@Entity
class Product implements ApplicationSettingSupport {

    private BigDecimal price;

    public BigDecimal getPriceIncludeVat() {
        BigDecimal vatRate = getApplicationSettingValue("VAT_RATE", BigDecimal.class);
        return price.multiply(vat).divided(BigDecimal.valueOf(100));
    }

    // throw UnSupportedOperationException
    @Override
    public <T> T getApplicationSettingValue(String settingKey, Class<T> bindingClass) {
        return super.getApplicationSettingValue(settingKey, bindingClass);
    }
    
}
```