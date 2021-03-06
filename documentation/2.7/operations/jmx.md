---
---
# Management and Monitoring using JMX <a name="JMX-Management-and-Monitoring"/>

 

## Introduction
As an alternative to the Ehcache Monitor, JMX creates a standard way of instrumenting classes and making them available to a management and monitoring infrastructure.

## Terracotta Monitoring Products
An extensive monitoring product, available in Enterprise Ehcache, provides a monitoring
server with probes supporting Ehcache-1.2.3 and higher for standalone and clustered
Ehcache. It comes with a web console and a RESTful API for operations integration.
See the [ehcache-monitor documentation](/documentation/2.7/operations/monitor) for more information. 
When using Ehcache 1.7 with Terracotta clustering, the Terracotta Developer Console
shows statistics for Ehcache.

## JMX Overview <a name="jmx"/>
JMX creates a standard way of instrumenting classes and making them
available to a management and monitoring infrastructure.
The `net.sf.ehcache.management` package contains MBeans and a `ManagementService` for JMX management of ehcache. It
is in a separate package so that JMX libraries are only required if you wish to use it - there is no leakage of JMX dependencies
into the core Ehcache package.

Use `net.sf.ehcache.management.ManagementService.registerMBeans(...)` static method
to register a selection of MBeans to the MBeanServer provided to the method.
If you wish to monitor Ehcache but not use JMX, just use the existing public methods on `Cache` and `CacheStatistics`.

The Management package is illustrated in the follwing image.

![Ehcache Image](/images/documentation/management_package.png) 

## MBeans <a name="MBeans"/>
Ehcache uses Standard MBeans. MBeans are available for the following:

* CacheManager
* Cache
* CacheConfiguration
* CacheStatistics

All MBean attributes are available to a local MBeanServer. The CacheManager MBean allows
traversal to its collection of Cache MBeans. Each Cache MBean likewise allows traversal to
its CacheConfiguration MBean and its CacheStatistics MBean.

## JMX Remoting <a name="JMXr-Remoting"/>
The Remote API allows connection from a remote JMX Agent to an MBeanServer via an <a id="MBeanServerConnection"></a>`MBeanServerConnection`.
Only `Serializable` attributes are available remotely. The following Ehcache MBean attributes are available remotely:

* limited CacheManager attributes
* limited Cache attributes
* all CacheConfiguration attributes
* all CacheStatistics attributes

Most attributes use built-in types. To access all attributes, you need to add ehcache.jar to the remote JMX client's
classpath e.g. `jconsole -J-Djava.class.path=ehcache.jar`.

## `ObjectName` naming scheme
* CacheManager - "net.sf.ehcache:type=CacheManager,name=&lt;CacheManager>"
* Cache - "net.sf.ehcache:type=Cache,CacheManager=&lt;cacheManagerName>,name=&lt;cacheName>"
* CacheConfiguration
- "net.sf.ehcache:type=CacheConfiguration,CacheManager=&lt;cacheManagerName>,name=&lt;cacheName>"
* CacheStatistics - "net.sf.ehcache:type=CacheStatistics,CacheManager=&lt;cacheManagerName>,name=&lt;cacheName>"

## The Management Service <a name="Management-Service"/>
The `ManagementService` class is the API entry point.
![Ehcache Image](/images/documentation/ManagementService.png) 
There is only one method, `ManagementService.registerMBeans` which is used to initiate JMX registration
of an Ehcache CacheManager's instrumented MBeans.
The `ManagementService` is a `CacheManagerEventListener` and
is therefore notified of any new Caches added or disposed and updates the MBeanServer appropriately.
Once initiated the MBeans remain registered in the MBeanServer until the CacheManager shuts down, at which time
the MBeans are deregistered. This behaviour ensures correct behaviour in application servers where applications are
deployed and undeployed.

<pre><code>
/**
* This method causes the selected monitoring options to be be registered
* with the provided MBeanServer for caches in the given CacheManager.
* 
* While registering the CacheManager enables traversal to all of the other
*  items,
* this requires programmatic traversal. The other options allow entry points closer
* to an item of interest and are more accessible from JMX management tools like JConsole.
* Moreover CacheManager and Cache are not serializable, so remote monitoring is not
* possible * for CacheManager or Cache, while CacheStatistics and CacheConfiguration are.
* Finally * CacheManager and Cache enable management operations to be performed.
* 
* Once monitoring is enabled caches will automatically added and removed from the
* MBeanServer * as they are added and disposed of from the CacheManager. When the
* CacheManager itself * shutsdown all registered MBeans will be unregistered.
*
* @param cacheManager the CacheManager to listen to
* @param mBeanServer the MBeanServer to register MBeans to
* @param registerCacheManager Whether to register the CacheManager MBean
* @param registerCaches Whether to register the Cache MBeans
* @param registerCacheConfigurations Whether to register the CacheConfiguration MBeans
* @param registerCacheStatistics Whether to register the CacheStatistics MBeans
*/
public static void registerMBeans(
   net.sf.ehcache.CacheManager cacheManager,
   MBeanServer mBeanServer,
   boolean registerCacheManager,
   boolean registerCaches,
   boolean registerCacheConfigurations,
   boolean registerCacheStatistics) throws CacheException {
</code></pre>

## JConsole Example <a name="JConsole-Example"/>
This example shows how to register CacheStatistics in the JDK platform MBeanServer, which
works with the JConsole management agent. 

<pre><code>
CacheManager manager = new CacheManager();
MBeanServer mBeanServer = ManagementFactory.getPlatformMBeanServer();
ManagementService.registerMBeans(manager, mBeanServer, false, false, false, true);
</code></pre>

CacheStatistics MBeans are then registered.
![Ehcache Image](/images/documentation/JConsoleExample.png) CacheStatistics MBeans in JConsole

## Hibernate statistics <a name="Hibernate-statistics"/>
If you are running Terracotta clustered caches as hibernate second-level cache provider, it is possible to access 
the hibernate statistics + ehcache stats etc via jmx.
`EhcacheHibernateMBean` is the main interface that exposes all the API's via jmx. It basically extends
two interfaces -- `EhcacheStats` and `HibernateStats`. And as the name implies `EhcacheStats` contains
methods related with Ehcache and `HibernateStats` related with Hibernate.
You can see cache hit/miss/put rates, change config element values dynamically -- like maxElementInMemory, TTI, TTL,
enable/disable statistics collection etc and various other things. Please look into the specific interface for more
details.

## JMX Tutorial <a name="JMX-Tutorial"/>
See this [online tutorial](http://weblogs.java.net/blog/maxpoon/archive/2007/06/extending_the_n_2.html).

## Performance
Collection of cache statistics is not entirely free of overhead.  In production systems where monitoring
is not required statistics can be disabled.  This can be done either programatically by calling
`setStatisticsEnabled(false)` on the cache instance, or in configuration by setting the `statistics="false"`
attribute of the relevant cache configuration element.
From Ehcache 2.1.0 statistics are off by default.
