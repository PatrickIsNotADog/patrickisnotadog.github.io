# Dependency Injection with Spring

In this post I will talk about the importance of connecting software components inside an application, how Dependency Injection emerged as a best practice and how you can wire components using the Spring framework.

## Java Framework for Enterprise Applications

Enterprise applications have to deal with complex problems, like distributed programming, transactions and security, which can be really difficult to handle when writing code from scratch, using plain Java. In order to provide a framework, which the developers will use to deal with these problems, Sun published Enterprise JavaBeans (EJB) at 1998. EJB tried to extend the notion of JavaBean components to the server side, providing extended enterprise services, but many engineers found that it lacked JavaBeans’ simplicity and it was hard to use.

As frequently happens in the Java world, another technology emerged as an alternative to EJBs, an open source framework, that provides all the enterprise features EJBs do, but also tries to reduce complexity and be more lightweight. Spring, initially released at 2002, is a Java framework and Inversion of Control container, that intends to make developing of applications easier.

## Wiring Components

One of the problems that has to be handled when developing an application, is how to wire  its components. Some definitions first: an application is a complete software solution, that addresses a specific problem and exposes some functionality. Of course there are more [specific definitions](https://en.wikipedia.org/wiki/Application_software), or others that divide applications to categories, but I think that the one above is pretty fine for the purpose of this post. Every application, except the really trivial ones, will contain more than one software components. When I say software component, I mean a piece of software, that provides a functionality and is generally written to be used inside an application’s context. A software component can be a simple class, following Object Oriented principles, or a set of classes, exposing a common API. Inevitably, associations will have to be created between components in every application, since this is the only way to offer rich service functionality, combining components’ individual behaviors.

The first and most intuitive way to link two components, is a Has-A relationship. I will use a phone book example in this post and specifically the class PhoneBook, which will expose a trivial findAllContacts() method, and Contacts, a Data Access Object, which will handle the connection with the persistent layer, where phone book’s contacts will be actually stored:

```java
public class PhoneBook {

    private Contacts contacts = new Contacts("/home/travelling/phonebook-example/contacts");

    public Object method findAllContacts() {
        return contacts.findAllContacts();
    }

}
```

In this example, I created a tight coupling between the classes `PhoneBook` and `Contacts`. This means that `PhoneBook` is highly dependent to `Contacts` and will make my life harder in a long term in several ways:

* If I want to change the file in which I save my contacts, I will have to change `PhoneBook`‘s source code too.
* If I want to use another implementation of `Contacts` (let’s say, save my contacts in a database instead of a file), I will have to change `PhoneBook`‘s source code.
* If I want to test `PhoneBook`‘s method `findAllContacts()`, I cannot use a mock `Contacts` object, to isolate `PhoneBook`. The `Contacts` object is created and initialized inside `PhoneBook`.

The first step to refine the code is to provide a `Contacts` interface, instead of a concrete class. I will use a `ContactsInterface` and a `ContactsFileImpl`, that will implement a Contacts [DAO](https://en.wikipedia.org/wiki/Data_access_object) , that reads from a file. So, if I want to change from file saved contacts to a MySQL database I would go from:

    private ContactsInterface contacts = new ContactsFileImpl("/home/travelling/phonebook-example/contacts");

to
 
    private ContactsInterface contacts = new ContactsMysqlImpl("localhost", 27001, "contacts");

## Dependency Injection

This is a little more elegant than the first approach, but still demands source changes in order to change Contacts implementation or provide a mock object. In order to solve this problem, I will use the [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)) design pattern, which assigns the task of creating a specific object’s field to another object, an assembler, that will populate the *field* object and link it to its *owner*. In the phone book example, `PhoneBook` will no longer need to care about how the `ContactsInterface` will be implemented or instantiated, but just provide the means for another object to initiate a `ContactsInterface` implementation, and pass it to `PhoneBook`.

We could say that there are three types of dependency injection. The first one is *Constructor Injection* and uses the constructor of the *owner* object, to set the *field* object. Under this approach `PhoneBook` would be transformed to:

```java
public class PhoneBook {

    private ContactsInterface contacts;

    public PhoneBook(ContactsInterface contacts) {
        this.contacts = contacts;
    }

    public Object method findAllContacts() {
        return contacts.findAllContacts();
    }

}
```

and the initialization could be done like:

```java
public class SpringApplication {

    public void main(String[] args) {
        ContactsInterface contacts = new ContactsFileImpl("/home/travelling/phonebook-example/contacts");
        PhoneBook phoneBook = new PhoneBook(contacts);
        phonebook.findAllContacts().stream().forEach(contact -> System.out.println(contact));
    }
}
```

The advantages of this approach are that:

* Since constructors are used to set *field* objects, we can be sure that all the fields that need to be created will actually be created and linked to the *owner* object. Or, in other words: no field that has to be initialized, will have a null value.
* We can assure that a field that has to be immutable, will actually be initialized once, and never change its value again. We simply do not provide a set method for this field.

On the other hand:

* A lot of parameters in the constructor could make the class look messy.
* If there are multiple ways to construct a valid object, it may be hard to show this through constructors, since constructors only differentiate themselves with the number and type of their parameters.
* If the constructor has multiple arguments of same type (e.g. String) it can be confusing to pass the right argument, based only on its position.
* Things can get awkward with multiple constructors and inheritance.

Another way to achieve Dependency Injection is *Setter* Injection, which requires the *owner* object to have set methods for all the fields that must be instantiated and the assembler/injector will use them to pass the created objects. In order for the `PhoneBook` to be used with a setter injector, it should be transformed to:

```java
public class PhoneBook {

    private ContactsInterface contacts;

    public PhoneBook() {
        super();
    }

    public setContacts(ContactsInterface contacts) {
        this.contacts = contacts;
    }

    public Object method findAllContacts() {
        return contacts.findAllContacts();
    }

}
```

and the initialization could be:

```java
public class SpringAplication {

    public void main(String[] args) {
        PhoneBook phoneBook = new PhoneBook();
        ContactsInterface contacts = new ContactsFileImpl("/home/travelling/phonebook-example/contacts");
        phoneBook.setContacts(contacts); phonebook.findAllContacts().stream().forEach(contact -> System.out.println(contact));
    }
}
```

You can see that the pros of the first approach are the second one’s cons and vice versa. There is no best way to achieve Dependency Injection, it’s up to the engineer to decide which one better fits his needs.

There is also third approach, called *Interface Injection*, but I will not refer to it in this post.

## What about Spring?

Spring controls the wiring, the configuring and the life cycle of the components using the Spring container, which contains beans, that represent the application’s components and the associations between them. There are two kinds of containers available to the developer:

* Bean factories, which are the simplest containers and provide basic support for dependency injection.
* Application contexts, which are extended bean factories, that also provide application framework services, such as: 
  * the ability to resolve textual messages from a properties file.
  * a generic way to load file resources, like images.
  * the ability to publish application events to interested event listeners.

Usually, developers choose `ApplicationContext` because of the extra features it provides, but here I will use an implementation of `BeanFactory`, `XmlBeanFactory`, to demonstrate an example. First, we need a `PhoneBook` class with a setter for its `ContactsInterface` field:

```java
public class PhoneBook {

    private ContactsInterface contacts;

    public PhoneBook() {
        super();
    }

    public setContacts(ContactsInterface contacts) {
        this.contacts = contacts;
    }

    public Object method findAllContacts() {
        return contacts.findAllContacts();
    }

}
```

Then we can create an `Application` class, which creates and configures the XmlBeanFactory in its main method:

```java
public class SpringApplication {

    public static void main(String[] args) {
        BeanFactory injector = new XmlBeanFactory(new ClassPathResource("spring.xml"));
        PhoneBook injectedPhoneBook = (PhoneBook) injector.getBean("phoneBook");
        List<String> contacts = injectedPhoneBook.getContacts().findAllContactNames();
        contacts.stream().forEach(contact -> System.out.println(contact));
    }

}
```

The main method above, creates an `XmlBeanFactory` using a spring.xml file in the classpath, and then gets a `PhoneBook` from it. Java 8’s stream and lamda functions are used then to print all contacts in the phone book.

The only thing that’s missing to fully understand what’s going on here, is the spring.xml file, which contains application’s beans, which are the components that must be created and wired: `PhoneBook` and `ContactsInterface` implementation. The spring.xml file is the below:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean id="contactsImpl" class="travelling.with.code.ContactsFileImpl" />

    <bean id="phoneBook" class="travelling.with.code.PhoneBook">
        <property name="contacts">
            <ref bean="contactsImpl"></ref>
        </property>
    </bean>

</beans>
```

Or if we want to use *Constructor Injection*:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean id="contactsImpl" class="travelling.with.code.ContactsFileImpl" />

    <bean id="phoneBook" class="travelling.with.code.PhoneBook">
        <constructor-arg>
            <ref bean="contactsImpl" />
        </constructor-arg>
    </bean>

</beans>
```

Finally, another great way to wire components in Spring is using Autowire:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean id="contactsImpl" class="travelling.with.code.FileContactsImpl" />

    <bean id="phoneBook" class="travelling.with.code.PhoneBook" autowire="autowire type" />

</beans>
```

## Further Reading

Great places to read more about Spring and Dependency Injection:

* Book: Manning, Spring in Action.
* Martin Fowler’s post: http://www.martinfowler.com/articles/injection.html

and also make sure you check Spring Boot!
