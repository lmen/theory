# How to implement an JAX-RS Method interceptor

The JAX-RS 2.0 specification defines the possibility to intercepts at different lifecycle points the request or the response handled by a JAX-RS container.

# How to implement an interceptor

The interception is performed by a class that implements one of or both interfaces:

- **javax.ws.rs.container.ContainerRequestFilter** : Has a method that is called when a resource method was found for the request url, according the JAX-RS matching rules, but *before* the resource method it is executed.
- **javax.ws.rs.container.ContainerResponseFilter** : has a method that is called *after* the resource method returns with response information.

The class of a filter must be annotated with **javax.ws.rs.ext.Provider** in order for the JAX-RS container to detect this filter implementation.

```java
@Provider
public class LogFilter implements ContainerRequestFilter {

    @Override
    public void filter( ContainerRequestContext containerRequest ) throws IOException {
         ...
    }
}
```

## How to stop the request processing

If *filter* method from **ContainerRequestFilter** calls **ContainerRequestContext.abortWith** then the request is no longer processed by the container and an HTTP response is return to the client.

Example:

 ```java
 ContainerRequestContext containerRequest;
 containerRequest.abortWith( Response.status( Status.UNAUTHORIZED ).build() );
 ```

## How to have access to the information about the resource method that is being filter

It does not exist a standard way to how this.

The JBoss Resteasy implementation provides this information througt a property called **org.jboss.resteasy.core.ResourceMethodInvoker**. The *filter* method from **ContainerRequestFilter** can use this property to find out this type of information.

Example:

```java
ContainerRequestContext containerRequest;
ResourceMethodInvoker methodInvoker = (ResourceMethodInvoker)containerRequest.getProperty( "org.jboss.resteasy.core.ResourceMethodInvoker" );

Method method = methodInvoker.getMethod();
```

# When is the filter class called ?

When a class only has the @Provider then this filter implementation will be called **in all requests**.

## Specifying a binding annotation

If the filter class as an annotation (called the binding annotation) that is itself annotated with  *javax.ws.rs.NameBinding* then the filter will only be applyed to the resource methods also annotated with the new defined annotation. The binding annotation can also be placed on a Resource class in order for the binding be applyed to all of its resource methods.

### Example

1. Defining the binding annotation

```java
@NameBinding
@Target( { ElementType.TYPE, ElementType.METHOD } )
@Retention( value = RetentionPolicy.RUNTIME )
public @interface Log {

}
```

2. Telling the JAX-RS container that this filter will be associated with @Log annotation.

```java
@Log
@Provider
public class LogFilter implements ContainerRequestFilter {
    ...
}

```

*Note:* The usage of the binding annotation, in this case @log, should not declare any values, since then it will restrict the binding to any resource method that also has the same annotation with the exact same values.

3. Apply on the REST Resource methods the binding annotation @Log in order to apply the filter.

```java
@Log
public Model doRestOperation( Model model ) {

}
```

## Binding annotation with required arguments.

What was described so far works well if the binding annotation does not have arguments that are mandatory, due to the binding matching algoritm used by the JAX-RS container. This matching algorithm can be replace with a new one specific for one application.

To do that, the filter implementation must be binded to the Resource Methods by a **javax.ws.rs.container.DynamicFeature** implementation.

An implementation of **javax.ws.rs.container.DynamicFeature** is called during the Resource Classes discovery phase, when the JAX-RS container starts, as it finds resources classes. During the discovery phase the **DynamicFeature** can analyse the resource method declaraction and register to the container a filter implementation for that resource method.

Example:

1. Define a *normal* annotation

```java
@Target( { ElementType.TYPE, ElementType.METHOD } )
@Retention( value = RetentionPolicy.RUNTIME )
public @interface LogAnnotation {

    String provider();

}
```

2. Define a **javax.ws.rs.container.DynamicFeature** class implementation

```java
@Provider
public class LogFilterBinder implements DynamicFeature {

    @Inject
    private LogFilter logFilter;

    @Override
    public void configure( ResourceInfo res, FeatureContext ctx ) {

        Method resourceMethod = res.getResourceMethod();

        if ( AnnotationFinder.find( resourceMethod, LogAnnotation.class ) != null ) {
            ctx.register( logFilter );
        }

    }
}
```

*Note I:* This class must be anotated with **javax.ws.rs.ext.Provider** in order for the JAX-RS container to discover this **DynamicFeature** implementation.

*Note II:* **ctx.register** needs an instance of a filter class. The filter instance can be retrieved from a CDI scope or from a the regular **new** operator.

3. Filter class implementation

```java
@Dependent
public class LogFilter implements ContainerRequestFilter {

    @Override
    public void filter( ContainerRequestContext containerRequest ) throws IOException {
        ...
    }
}
```

*Note:* This class is implemented as it was in the section above, but without beging anotated with **javax.ws.rs.ext.Provider**, since it is the *DynamicFeature* that registers the filter instance.

4. Annotation usage

```java
@Log(provider="com.company.operation")
public Model doRestOperation( Model model ) {

}
```