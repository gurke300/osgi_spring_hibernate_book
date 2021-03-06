# Hibernate In OSGi
In this chapter we are going to prepare our environment to be able to utilize the Hibernate
engine.
We will enhance the PaxExamProvisioningSupport class to provide the container with the dependencies.
At least we will configure the Apache Felix OSGi container to solve some dependency resolution problems.

## Provisioning the OSGi Container
The Hibernate engine is a powerful tool for communicating with a relational database and with that it comes with several dependencies as:

 * a bytecode generator for its powerful proxy mechanism
 * logging facilities
 * xml processing
 * and some javaee interfaces due to its JPA compliance

Lets have a look on our +PaxExamProviosioningSupport+ class which lists the new bundles:
[source, java]
----
[...]
 public static Option jbossBundles() {                                          <1>
        return CoreOptions.composite(
            mavenBundle("org.jboss.logging", "jboss-logging", "3.1.3.GA"),
            mavenBundle("org.jboss", "jandex", "1.2.1.Final"));
    }

    public static Option xmlBundles() {                                         <2>
        return CoreOptions.composite(
            mavenBundle("com.fasterxml", "classmate", "1.1.0"),
            mavenBundle("org.apache.servicemix.specs", "org.apache.servicemix.specs.stax-api-1.2"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.dom4j", "1.6.1_5")
        );
    }

    public static Option hibernateBundles() {                                   <3>
        return CoreOptions.composite(
            mavenBundle("org.javassist", "javassist", "3.18.2-GA"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.ehcache", "2.6.9_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.antlr", "2.7.7_3"),
            mavenBundle("org.hibernate.javax.persistence", "hibernate-jpa-2.1-api", "1.0.0.Final"),
            mavenBundle("org.hibernate", "hibernate-core", "4.3.8.Final"),
            mavenBundle("org.hibernate", "hibernate-entitymanager", "4.3.8.Final"),
            mavenBundle("org.hibernate.common", "hibernate-commons-annotations", "4.0.5.Final"),
            mavenBundle("org.hibernate", "hibernate-ehcache", "4.3.8.Final")
        );
    }
[...]
----

<1> the jboss dependencies used inside hibernate
<2> the xml processing related dependencies
<3> the hibernate bundles and some additional closely related ones. At a later point it might be useful to rearrange this block into smaller groups.

The simple test class looks like this:
[source, java]
----
@RunWith(PaxExam.class)
public class RunOSGiWithHibernateProvisioningSupportIT {

    @Configuration
    public Option[] configureTest() throws IOException {

        return CoreOptions.options(
            CoreOptions.cleanCaches(),
            aopAllianceBundle(),
            PaxExamProvisioningSupport.javaeeBundles(),
            PaxExamProvisioningSupport.xmlBundles(),
            PaxExamProvisioningSupport.jbossBundles(),
            PaxExamProvisioningSupport.hibernateBundles(),
            PaxExamProvisioningSupport.slf4jBundles(),
            CoreOptions.junitBundles());
    }

    @Test
    public void shouldContainTheHibernateFrameworkBundles() throws Exception {
        Bundle hibernateCoreBundle = FrameworkUtil.getBundle(org.hibernate.Session.class);
        Assert.assertNotNull(hibernateCoreBundle);
        Assert.assertEquals(Bundle.ACTIVE, hibernateCoreBundle.getState());
    }
}
----

The output of running this test says:

[source, console]
----
org.osgi.framework.BundleException: Uses constraint violation. Unable to resolve bundle revision org.hibernate.core [27.0] because it is exposed to package 'javax.xml.stream' from bundle revisions org.apache.felix.framework [0] and org.apache.servicemix.specs.stax-api-1.2 [19.0] via two dependency chains.

Chain 1:
  org.hibernate.core [27.0]
    import: (osgi.wiring.package=javax.xml.stream)
     |
    export: osgi.wiring.package=javax.xml.stream
  org.apache.felix.framework [0]                                                    <1>

Chain 2:
  org.hibernate.core [27.0]
    import: (osgi.wiring.package=org.dom4j)
     |
    export: osgi.wiring.package=org.dom4j; uses:=javax.xml.stream
  org.apache.servicemix.bundles.dom4j [20.0]
    import: (&(osgi.wiring.package=javax.xml.stream)(version>=1.0.0)(!(version>=2.0.0)))
     |
    export: osgi.wiring.package=javax.xml.stream
  org.apache.servicemix.specs.stax-api-1.2 [19.0]                                   <2>
----

<1> the system bundle exporting javax.xml.stream package in the version _0.0.0.1_008_JavaSE_
<2> the stax-api bundle exporting javax.xml.stream package in the version _1.2_

Mmmmh. Obviously we have a problem with the resolution of the _javax.xml.stream_ dependency. The reason for that is based on how the bundle descriptor of _hibernate-core_ declares its *_Import-Package_* header for the javax.xxx dependencies.
It is missing a version number. In contrast to the *_Import-Package_* header of the _org.dom4j_ library that declares _javax.xml.stream;version="[1.0,2)"_.

In order to solve this we have several options.

* adjust the export packages of the system bundle
* adjust the *_Import-Packages_* header within hibernate-xxx bundle to resolve to a specific _javax.xml_ version
* adjust the _dom4j_ bundle to import the system bundle's packages.

The first options require us to know how *Apache Felix* bootstraps the OSGi container with the system bundle (see *Chapter 2*).
If you look inside the *Apache Felix* jar resource you will find a file named *default.properties*. In this file *Apache Felix* lists all packages grouped by the target Java VM it will run on. The version of all exported bundles is set to
_0.0.0.1_some java classifier_.

Now the bundle _hibernate-core_ is linked to the system bundle's _javax.xml.stream_ export and the _org.dom4j_ bundle is linked the _stax-api_ bundle. Because of _hibernate-core_ that is linked to the _org.dom4j_ bundle the OSGi framework detects a mismatch because there are the same classes loaded by 2 different classloaders (system bundle's classloader, and _stax-api_ bundle's classloader). If the framework would load the bundle, you would face a _java.lang.LinkageError_ at runtime.

The solution of this situation is to make the _org.dom4j_ and the _hibernate-core_ bundle resolving to the same _javax.xml.stream_ exporting bundle. In order to do this we need to change the *_default.properties_* file so copy it into your classpath.
Basically there are two options:

. change the export version of the _javax.xml.stream_ related packages to _1.0.0.1_xxxx_
    .. remove the _stax-api_ bundle from the +PaxExamProvisioningSupport+ class
. delete the _javax.xml.stream_ related exports from the JDK runtime section in *_default.properties_* file
    .. in that case you have to keep the _stax-api_ bundle.

For option 1 you need to remove the _stax-api_ bundle from the _xmlBundles_ function:

----
[...]
    public static Option xmlBundles() {
        return CoreOptions.composite(
            mavenBundle("com.fasterxml", "classmate", "1.1.0"),
            // mavenBundle("org.apache.servicemix.specs", "org.apache.servicemix.specs.stax-api-1.2"),                            <1>
            mavenBundle("org.apache.servicemix.bundles", org.apache.servicemix.bundles.dom4j", "1.6.1_5"));
    }
[...]
----

<1> the _stax-api_ bundle has been commented out

Extract of the *_default.properties_* file:

----
[...]

jre-1.8=, \
 [...]
 javax.xml.stream.events;uses:="javax.xml.namespace,javax.xml.stream";version="1.0.0.1_008_JavaSE", \
 javax.xml.stream.util;uses:="javax.xml.stream,javax.xml.stream.events,javax.xml.namespace";version="1.0.0.1_008_JavaSE", \
 javax.xml.stream;uses:="javax.xml.stream.events,javax.xml.namespace,javax.xml.stream.util,javax.xml.transform";version="1.0.0.1_008_JavaSE", \
 javax.xml.transform.dom;uses:="javax.xml.transform,org.w3c.dom";version="0.0.0.1_008_JavaSE", \
 javax.xml.transform.sax;uses:="org.xml.sax.ext,javax.xml.transform,org.xml.sax,javax.xml.transform.stream";version="0.0.0.1_008_JavaSE", \
[...]
----

When option 2 is chosen, you just need to remove the lines from the *_default.properties_* file.

[CAUTION]
.Caution by deleting export packages
=====================================================================
If you take a closer look to the export package listing you will see
a _uses_ directive on some packages. So it might be the case that you
end up with removing more export packages than intended.
Another drawback is that you might need to add other bundles to export
those removed packages again, because of other bundles requiring them.
=====================================================================



== The Hibernate OSGi Project ==
* Hibernate 4 Services via java.util.ServiceLoader
* Transactions
* the jta transaction manager

== Classpath Considerations ==

**ClassCastException: org.dom4j.DocumentFactory cannot be cast to org.dom4j.DocumentFactory**

[source,console]
----
Caused by: org.hibernate.InvalidMappingException: Unable to read XML
	at org.hibernate.internal.util.xml.MappingReader.legacyReadMappingDocument(MappingReader.java:375)
	at org.hibernate.internal.util.xml.MappingReader.readMappingDocument(MappingReader.java:304)
	at org.hibernate.cfg.Configuration.add(Configuration.java:516)
	at org.hibernate.cfg.Configuration.add(Configuration.java:512)
	at org.hibernate.cfg.Configuration.add(Configuration.java:686)
	at org.hibernate.cfg.Configuration.addInputStream(Configuration.java:724)
	at org.springframework.orm.hibernate4.LocalSessionFactoryBean.afterPropertiesSet(LocalSessionFactoryBean.java:344)
	at com.ottogroup.expo.hibernate.ExpoLocalSessionFactoryBean.afterPropertiesSet(ExpoLocalSessionFactoryBean.java:67)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:483)
	at org.apache.aries.blueprint.utils.ReflectionUtils.invoke(ReflectionUtils.java:297)
	at org.apache.aries.blueprint.container.BeanRecipe.invoke(BeanRecipe.java:958)
	at org.apache.aries.blueprint.container.BeanRecipe.runBeanProcInit(BeanRecipe.java:712)
	... 39 more
Caused by: org.dom4j.DocumentException: org.dom4j.DocumentFactory cannot be cast to org.dom4j.DocumentFactory Nested exception: org.dom4j.DocumentFactory cannot be cast to org.dom4j.DocumentFactory
	at org.dom4j.io.SAXReader.read(SAXReader.java:484)
	at org.hibernate.internal.util.xml.MappingReader.legacyReadMappingDocument(MappingReader.java:325)
[...]
----

This issue has its origin in the way the dom4j lib is bootstrapping its internal *DocumentFactory*. If you look into the *DocumentFactory* source you will find a tiny member
called "SingletonStrategy" that controls the way the *DocumentFactory* is loaded. The default one is the SimpleSingleton that instantiate the DocumentFactory using the
"Thread.currentThread().getContextClassLoader()" which is undefined within the OSGi world.

To solve this issue you need to change the thread's ContextClassLoader e.g.:
```
// setting the ContextClassLoader for the lib used within hibernate solves the problem
ClassLoader currentThreadContextCL = Thread.currentThread().getContextClassLoader();
try {
    ClassLoader classLoader = this.getClass().getClassLoader();
    Thread.currentThread().setContextClassLoader(classLoader);
	// doSomething
    } finally {
        Thread.currentThread().setContextClassLoader(currentThreadContextCL);
    }
```
