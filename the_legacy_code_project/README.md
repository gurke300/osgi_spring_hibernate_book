# The Legacy Code Project
The starting point of the migration to an OSGi friendly form is an platform application that has been developed almost since more than 6 years by several developers. Such projects often come up with a monolithic and technology driven (in opposite to a use case driven) structure meaning that you will find packages like:

* com.company.application.persistence.dao
 * containing interfaces for the data access objects used for the CRUD operations of the domain entities
* com.company.application.app.persistence.dao.hibernate
 * holds the hibernate specific DAO implementation classes (with JPA the EntityManager is doing this job)
* com.company.application.domain
 * holding the domain objects
* com.company.application.utils
 * there are always utility classes :-)
* com.company.application.services
 * services does usually orchestrate several DAOs, or connect to other systems like search, payment, etc.
* com.company.application.business
 * a package that holds certain kinds of business related logic. In an ecommerce project you may find price/discount calculation, filtering of products, etc. as typical use cases

Because Spring is used as the DI Framework there are direct dependencies within the code like implemented *InitializingBean*s, *FactoryBean*s. And because Spring's excellent support for 3rd party frameworks such as [Hibernate](http://hibernate.org) of course there are direct dependencies to the *HibernateTemplate* or *HibernateDaoSupport*, etc. The Spring configuration is done using the XML approach. So you have the bean definitions in files like:
* spring-root-configuration.xml basically importing the other spring configuration files as classpath resources
* spring-persistence-configuration.xml
 * holds the configuration of the hibernate SessionFactory using Spring's LocalSessionFactoryBean
* spring-dao-configuration.xml
 * defining all DAO implementations
* spring-entities-configuration.xml
 * defines domain entities with configured business logic that are merged with the beans loaded by hibernate (will be shown later in the book)
* spring-transactions-configuration.xml
 * defines declarative transactions using springs AOP support
* spring-services-configuration.xml
* ...

Clients usually know all the internals and can overwrite bean definitions with custom ones.
That makes development under backward compatibility requirements to a real challenge.

## Diving Into The Details
In this chapter we will check for some specific implementation details, that might challenges us. Some aspects of the API we will check is:
* The Domain model
* Collection of spring configured beans
    * this feature is basically for collecting object instances configured across different spring configuration files. Basically this API was developed to hook in the configuration of Hibernate (especially cache setup) but was used later to collect arbitrary objects like properties, too.
* configuration of the Hibernate SessionFactory
    * Transaction Configuration
    * Auto Merging of spring configured beans with object instances delivered by Hibernate using Hibernate's Interceptor API.
    * Custom User Types




