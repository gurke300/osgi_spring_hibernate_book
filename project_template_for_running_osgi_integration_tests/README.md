# Project Template For Running OSGi Integration Tests
I don't want to explain why unit testing is important. In my opinion testing is a standard in software development so it is unnecessary to talk about its need.
When you use frameworks like Hibernate, Spring, OSGi, Servlets, etc. it becomes essential to prove that you use these frameworks in the intended way. To say it in other words: if you work with Spring you need a test that bootstraps the application context to prove that your bean definitions are working as well as you need a test in which you prove that your Hibernate configurations/mappings are working.
In that sense you also need a test proving that your piece of software runs properly in an OSGi container.

For Maven based development there is PAX Exam which does the job of provisioning and running an OSGi container. I strongly recommend to check the [PaxExam 4 documentation](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Getting+Started+with+OSGi+Tests) which is already very good for Open Source projects. But especially the
[PaxExam 4 Maven dependency section](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Maven+Dependencies) unfortunately does not mention the specific dependencies you need for running integration tests against an OSGi container. So I had to grap the [Pax Exam Sources](https://github.com/ops4j/org.ops4j.pax.exam2) to find out what I would really need. Besides that a lot of articles are written around older versions of PaxExam, so their benefit is a bit limited if you use newer versions.
That's why the next sections will describe how you can write JUnit Integration Tests with PaxExam 4.

## The First Integration Test Using Pax Exam 4
First we setup a project in our preferred IDE. I am using Netbeans because it has a nice integration of Maven. But it should work in IntelliJ and Eclipse the same way. You may also check out the sources from Github here.
> <span style="color:red">TODO: add link to Github projects</span>

### The First Simple Test
In here the package imports are included just to show from where those classes are imported from. Later we will only list them if they have not been mentioned before or if we may have classes with the same name within the classpath (e.g. an *@Configuration* annotation also exists in the *Spring Framework*)
```
package com.mycompany.osgi.paxexam.it;

import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertTrue;

import java.io.IOException;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.ops4j.pax.exam.Configuration;
import org.ops4j.pax.exam.CoreOptions;
import org.ops4j.pax.exam.Option;
import org.ops4j.pax.exam.junit.PaxExam;
import org.osgi.framework.BundleContext;

import javax.inject.Inject;

/**
 *
 * @author Stephan
 */
@RunWith(PaxExam.class)
public class RunOSGiContainerIT {

    @Inject
    private BundleContext bundleContext;

    @Configuration
    public Option[] configure() throws IOException {
        return CoreOptions.options(
            CoreOptions.junitBundles()
        );
    }

    @Test
    public void shouldHaveInjectedTheBundleContextOfThisTestBundle() {
        assertNotNull(this.bundleContext);
        assertTrue("JUnit Test Bundle should start with PAX", this.bundleContext.getBundle().getSymbolicName().startsWith("PAX"));
    }

}
```
Basically we don't do anything in here except checking that we are integrated to an OSGi platform due to the check that we got our bundle's *org.osgi.framework.Bundlecontext*. That is quite similar to a *javax.servlet.ServletContext* providing you an entry point to the platform's environment.

How does this work?
Pax Exam will create a bundle of your test in the background. It will start up the OSGi container using the provided configuration options and will deploy the test bundle into the OSGi container. Besides that it will take care of resolving the variables marked with inject.

Below you can find the pom for our first running OSGi Integration Test Example Project.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany.osgi.test</groupId>
    <artifactId>PAXExamIT</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <name>PAXExamIT OSGi Bundle</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.osgi</groupId>
            <artifactId>org.osgi.core</artifactId>
            <version>5.0.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.servicemix.bundles</groupId>
            <artifactId>org.apache.servicemix.bundles.javax-inject</artifactId>
            <version>1_2</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.ops4j.pax.exam</groupId>
            <artifactId>pax-exam-junit4</artifactId>
            <version>4.2.0</version>
            <scope>test</scope>
        </dependency>
        <!--
            pax-exam-container-native will run the OSGi platform.
            In our case its the felix framework defined within the profile section.
        -->
        <dependency>
            <groupId>org.ops4j.pax.exam</groupId>
            <artifactId>pax-exam-container-native</artifactId>
            <version>4.2.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.ops4j.pax.exam</groupId>
            <artifactId>pax-exam-link-mvn</artifactId>
            <version>4.2.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.17</version>
                <configuration>
                    <includes>
                        <include>%regex[.*IT.*]</include>
                    </includes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <profiles>
        <profile>
            <id>felix</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <dependencies>
                <dependency>
                    <groupId>org.apache.felix</groupId>
                    <artifactId>org.apache.felix.framework</artifactId>
                    <version>4.4.1</version>
                    <scope>test</scope>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
</project>
```
On my way to get OSGi integration tests using PaxExam working I found some gaps in the documentation.
So lets walk through the dependencies step by step and clarify for what they are good for.
* **General Project dependencies**
 * *org.osgi.core* is needed in order to use the OSGi API.
  * It is like the javax.servlet library that contains the API classes (usually Java interfaces) you will implement.
 * *org.apache.servicemix.bundles.javax-inject* is the javax-inject api wrapped as bundle
   * this kind of dependency will be used a lot in this book. A lot of older java libraries are not build for running within an OSGi environment. They are simply missing the specific OSGi headers within the Manifest.mf file, that is part of a *Java Archive*. Those headers are added when those libraries are repackaged again. And this is one candidate of this kind of dependencies. This dependency is just included to use *@javax.inject.Inject* annotations within our test class, because *PaxExam* is able to inject those dependencies.
* **Test Dependencies** next to JUnit 4, that should be clear
    * *pax-exam-junit4* is the integration part for JUnit4 based tests. It defines the *PaxExam.class* as JUnit Runner that will take over all the bootstrapping and integration to a test container (in fact you can write JavaEE Integration tests with PaxExam, too)
    * *pax-exam-container-native* this dependency is needed for loading the OSGi framework. In order to do so it uses the **java.util.ServiceLoader** to look for an implementation of the *org.osgi.framework.launch.FrameworkFactory*.
    * *pax-exam-link-mvn* is used as dependency in order to provision (configuring the OSGi platform with the appropriate bundles). We will use this feature later in order to configure the OSGi container with our maven dependencies.
* **The Profile Section**
 * in here we configured a concrete implementation of the [Apache Felix Framework](http://felix.apache.org) later we will add another OSGi platform just to prove specification compliance. How this framework is initialized we will see later as we will run into some dependency resolution problems.

## Enhancing Testability Providing Composite Options
If you are developing several bundles each tested separatly you might want to create some preconfigured provisioning options, so you don't have to list each bundle in the test classes again.

E.g. to have the spring bundles right in place you can create a class in which you can define a *CompositeOption*.
```
public class PaxExamProvisioningSupport {

    /**
     * Returns the bundle for the aop interface of the aopalliance needed e.g. by spring-aop.
     * @return bundle for aop domain
     */
    public static Option aopAllianceBundle() {
        return mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.aopalliance", "1.0_6");
    }

    /**
     * Returns the spring bundles used in this project.
     * @return spring 4 bundles
     */
    public static Option springBundles() {
        return CoreOptions.composite(
            // spring dependencies bundles
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-aop", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-beans", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-context", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-context-support", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-core", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-jdbc", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-orm", "4.0.7.RELEASE_1"),
            mavenBundle("org.apache.servicemix.bundles", "org.apache.servicemix.bundles.spring-tx", "4.0.7.RELEASE_1"));
    }
}
```
And using it within your OSGi test.
```
import static com.mycompany.osgi.paxexam.it.PaxExamProvisioningSupport.aopAllianceBundle;
import static com.mycompany.osgi.paxexam.it.PaxExamProvisioningSupport.springBundles;

import org.osgi.framework.FrameworkUtil;
[...]

@RunWith(PaxExam.class)
public class RunOSGiWithProvisioningSupportIT {

    @Inject
    private BundleContext bundleContext;

    @Configuration
    public Option[] configureTest() throws IOException {

        return CoreOptions.options(
            CoreOptions.cleanCaches(),
            aopAllianceBundle(),
            springBundles(),
            CoreOptions.junitBundles());
    }

    @Test
    public void shouldEnableUsToInspectTheOSGiFrameworkUsingTheGoGoShell() throws Exception {
        Assert.assertNotNull(this.bundleContext);
        Assert.assertNotNull(FrameworkUtil.getBundle(org.springframework.context.ApplicationContext.class));
    }

}
```
In that way you only need to define the required bundles once and you can keep your configuration section as small as possible. The FrameworkUtil class is provided by the *osgi core api*. And as you can see you can get inter-bundle access. If we come to the configuration of hibernate we will use this class to modify the classloader.

If you run this test the console should log something like:
```
[...]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.geronimo.specs.geronimo-atinject_1.0_spec]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.aopalliance]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-aop]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-beans]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-context]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-context-support]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-core]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-jdbc]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-orm]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.servicemix.bundles.spring-tx]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [PAXEXAM-PROBE-50624a73-9187-4190-a258-357737fd697d]
[...]
```
Two interesting points from the output may be these two lines:
```
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [org.apache.geronimo.specs.geronimo-atinject_1.0_spec]
org.ops4j.pax.logging.pax-logging-api[org.ops4j.pax.swissbox.extender.BundleWatcher] : Scanning bundle [PAXEXAM-PROBE-50624a73-9187-4190-a258-357737fd697d]
```
The first bundle is "injected" by the pax exam framework in order to resolve the *@javax.inject.Inject* annotation within the test. The second bundle is the one the pax exam framework creates automatically from your test. In that way your test classes becoming runable OSGi bundles.

[![Build Status](https://www.gitbook.io/button/status/book/gurke300/osgi-with-spring-4-and-hibernate-4)](https://www.gitbook.io/book/gurke300/osgi-with-spring-4-and-hibernate-4/activity)
