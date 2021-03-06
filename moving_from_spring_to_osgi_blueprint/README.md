# Moving from Spring to OSGi Blueprint
OSGi Blueprint was defined in the OSGi Enterprise specification. It has its roots in the Spring Dynamic Modules project and that's the good news: If you know Spring then OSGi Blueprint will look very familiar to you.
Allthough OSGi Blueprint is not that powerful as it lacks some possibilities in defining beans (e.g. no *abstract* or *parent* beans definitions) it is a great option to bootstrap a bundle's logic and interact with the OSGi container using services.

OSGi Blueprint defines some mechanisms to extend the container's behaviour. So you can define your own *TypeConverter*s to bind bean definitions to certain types. As known from Spring you can define custom namespacehandler to enhance xml configuration options for application specific components.

In this section we are going to see how we can use OSGi Blueprint to define beans/services and how we can interact with the Blueprint container. It is highly recommended to also take a look into the OSGi Blueprint Specification which is very well written.

In the first X sections some basics will be shown to prepare for moving from Spring to OSGi Blueprint.

## Choosing An OSGi Blueprint Implementation
As OSGi just specifies the Blueprint API you are going to look for an implementation. Currently there are two (both OpenSource) Blueprint implementations:
* [Apache Aries](http://aries.apache.org/) Blueprint and
* [Eclipse Gemini Blueprint](http://www.eclipse.org/gemini/blueprint/documentation/reference/1.0.2.RELEASE/html/index.html).

### Eclipse Gemini Blueprint
Eclipse Gemini Blueprint has its origins in the Spring Dynamic Module project and has a very good spring support.
The Eclipse Gemini Blueprint implementation goes further than the OSGi Blueprint specification and integrate Spring capabilities into OSGi. E.g. you can use Spring's *Aware*-interfaces or autowiring capabilities. The list of extensions compared to the specification can be read [in the Eclipse Gemini Blueprint documentation](http://www.eclipse.org/gemini/blueprint/documentation/reference/1.0.2.RELEASE/html/blueprint.html)

But there is a big disadvantage. In its current Milestone release version (2.0.0 M2) it is bound to Spring 3.x.
Allthough Eclipse Gemini Blueprint makes it easier to use Spring based applications within OSGi it is bound to Spring 3.

Because we are going to use Spring 4 we will configure our application using Apache Aries Blueprint implementation.

### Apache Aries Blueprint
*Apache Aries Blueprint* in contrast to *Eclipse Gemini Blueprint* is a less feature rich *Blueprint Specification* implementation. It has support for
* managed properties,
* property replacement like Spring and
* declarative transactions.

Also the capabilities of binding bean definitions to object instances are not that powerful as known from *Spring/Eclipse Gemini Blueprint*. E.g. *Apache Aries Blueprint* cannot bind an A.class to a property out of the box. Instead will try to put in an instance of class A (*A.class.newInstance(...)* later we will see how to change that).

But *Apache Aries Blueprint* implementation is lightweight and comes with lesser dependencies then *Eclipse Gemini Blueprint* And with that you do not need to provision your OSGi container with Spring dependencies if you don't use them.


In order to use the Apache Aries Blueprint implementation we will extend our provisioning support class for our PaxExam integration test.
```
public static Option apacheAriesBlueprintBundles() {
        return CoreOptions.composite(
            mavenBundle("org.apache.aries.blueprint", "org.apache.aries.blueprint.core"),
            mavenBundle("org.apache.aries.blueprint", "org.apache.aries.blueprint.cm"),
            mavenBundle("org.apache.aries.blueprint", "org.apache.aries.blueprint.annotation.api"),
            mavenBundle("org.apache.aries.proxy", "org.apache.aries.proxy.api", "1.0.0"),
            mavenBundle("org.apache.aries.proxy", "org.apache.aries.proxy.impl", "1.0.4"),
            mavenBundle("org.apache.aries.quiesce", "org.apache.aries.quiesce.api", "1.0.0"),
            mavenBundle("org.apache.aries", "org.apache.aries.util", "1.1.0")
        );
    }

    public static Option slf4jBundles() {
        return CoreOptions.composite(
            mavenBundle("org.slf4j", "slf4j-api", "1.7.7"),
            // a fragment cannot be started but installed
            mavenBundle("org.slf4j", "slf4j-simple", "1.7.7").noStart());
    }

```
Because the Apache Aries Blueprint implementation has dependencies to [slf4j](http://slf4j.org/) we need those bundles too.

<div style="background-color:#EAEAFF;border:solid #6A6AFF 1px; margin:10px; padding:10px; ">
**Note:** <br/><br/>
The slf4j-impl bundle is a fragment. A fragment is a special osgi bundle / resource with some differences regarding the lifecycle. The Fragment-Bundle will enhance the classpath of the *Host-Bundle*. In that way it can introduce new resources. But its lifecycle is bound to the one of the *Host-Bundle*. So for a fragment bundle the OSGi container is not creating a dedicated classloader, but the class loader of the *Host-Bundle* will have access to the Fragment-Bundle's resources.
</div>

If you run a first test with these additional bundles installed you should see these lines in your console:
```
[...]
[FelixShutdown] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.aries.blueprint.cm
[FelixShutdown] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.aries.blueprint.core
[...]
```
Now we are ready to use Blueprint to define our beans.

## Using Blueprint - A Simple Showcase
OSGi Blueprint uses xml files in which application logic is declarativly listed. These xml files are supposed to be located under
*OSGI-INF/blueprint/* by default. The Blueprint Extender Service will look for all xml files under this location and will create a Blueprint Container which corresponds to Spring's application context.
Here is the first blueprint configuration.
```
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.osgi.org/xmlns/blueprint/v1.0.0                   http://www.osgi.org/xmlns/blueprint/v1.0.0/blueprint.xsd" 
    default-availability="mandatory">

    <bean id="helloBean" class="com.mycomp.shop.HelloBean"/>
    <service ref="helloBean" interface="com.mycomp.shop.IHelloBean"/>

</blueprint>
```
In the example above we declared a bean called HelloBean and we published it to the OSGi container as a service.
And that is a strength of OSGi Blueprint. It makes it easy to publish services or consume OSGi Services from the OSGi container.

Also Spring knows *Bean*s and *Service*s (actually Springs *Service*s are Beans marked via the *@Service* annotation).
But actually there is not much of a difference between a *Bean* and a *@Service* in Spring. Both are known across the application context. The *@Service* annotation provides the possibility to distinguish it from general beans.

In OSGi Blueprint there is a huge difference between a *Bean* and a *Service*. OSGi is all about Services and their dynamics.
If you want to provide a service to the container's environment you declare services via the &gt;service/&lt; element. Whereas a &gt;bean/&lt; as just locally visible within the Blueprint container.

And there is something more we get using Blueprint.
```
@RunWith(PaxExam.class)
public class RunOSGiWithProvisioningSupportIT {

    @Inject
    private IHelloBean helloBean;

    @Inject
    private BlueprintContainer blueprintContainer;

    @Inject
    private BundleContext blueprintBundleContext;

    @Configuration
    public Option[] configureTest() throws IOException {

        return CoreOptions.options(
            CoreOptions.cleanCaches(),
            aopAllianceBundle(),
            springBundles(),
            PaxExamProvisioningSupport.apacheAriesBlueprintBundles(),
            PaxExamProvisioningSupport.slf4jBundles(),
            CoreOptions.bundle("reference:file:target/classes"),
            CoreOptions.junitBundles());
    }

    @Test
    public void shouldPrintTheBeanDefinition() throws Exception {

        Assert.assertNotNull(this.blueprintContainer);
        Assert.assertNotNull(this.helloBean);
        helloBean.hello();
        Optional<? extends BeanMetadata> beanMetadata = this.blueprintContainer.getMetadata(BeanMetadata.class).stream().filter(bm -> {
            System.out.println("bm.getId() "+bm.getId());
            return bm.getId().equals("helloBean");
        }).findAny();
        Assert.assertTrue(beanMetadata.isPresent());
        IHelloBean tmpHelloBean = (IHelloBean)this.blueprintContainer.getComponentInstance(beanMetadata.get().getId());
        System.out.println(tmpHelloBean.hello());
    }
}
```
If you look at this test case you can see that beside our defined beans, Blueprint is publishing some more for us.
Next to our *helloBean* there is the
* *BlueprintContainer* that allows us to interact with it (listing bean definitions or looking up bean instances)
* the second one is the *BundleContext* which comes via its blueprint bean name *blueprintBundleContext*.

In the code example you can see how we interact with the *BlueprintContainer*. With
```
BlueprintContainer.getMetadata(Class<T extends ComponentMetadata>)
```
you can get access to the
* *bean* definitions via BeanMetadata.class
* *service* definitions via ServiceMetadata.class
* *service references* definitions via ServiceReferenceMetadata.class

<div style="background-color:#EAEAFF;border:solid #6A6AFF 1px; margin:10px; padding:10px; ">
**Note:** <br/><br/>
In the OSGi specification you will find the naming of **Manager* elements like *BeanManager* or *ServiceManager* which are the handler for the *&lt;bean/&gt;* or *&lt;service&gt;* elements.
</div>

Via the *BundleContext* you can get access to resources within your bundle or you can interact with the OSGi container.

So OSGi Blueprint gives you the possibility to create bundles that can be bootstrapped without an *Activator*.

* Type Converter

Type Conversion in Apache Aries
Collecting Beans from a Blueprint Container
Declarative Transactions in Apache Aries

## Extending The Type Conversion With Custom TypeConverters
Compared to Spring, Apache Aries blueprint implementation is not that powerful if it comes to automatic type conversion.

To be able to extend the type conversion capabilities of the Blueprint Container the OSGi Blueprint Spec defines a mechanism called *TypeConverter*. TypeConverter are needed for

```
import org.osgi.framework.BundleContext;
import org.osgi.framework.FrameworkUtil;
import org.osgi.service.blueprint.container.Converter;
import org.osgi.service.blueprint.container.ReifiedType;

public class ClassInstanceCreater implements Converter {

    private BundleContext bundleContext;

    @Override
    public boolean canConvert(Object sourceObject, ReifiedType targetType) {

        System.out.println("SourceObject -> "+sourceObject+" reified type "+targetType);

        return String.valueOf(sourceObject).endsWith(".class");
    }

    @Override
    public Object convert(Object sourceObject, ReifiedType targetType) throws Exception {

        String sourceClass = sourceObject.toString();

        String pureClassName = sourceClass.substring(0, sourceClass.lastIndexOf(".class"));

        System.out.println("Extracted Classname is -> "+pureClassName);

        Class<?> clazz = FrameworkUtil.getBundle(Session.class).loadClass(pureClassName);
        System.out.println("Resolved Class -> "+clazz);
        return clazz;
    }

    public BundleContext getBundleContext() {
        return bundleContext;
    }

    public void setBundleContext(BundleContext bundleContext) {
        this.bundleContext = bundleContext;
    }

}
```
</div>
