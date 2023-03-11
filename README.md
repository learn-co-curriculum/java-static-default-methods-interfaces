# Interfaces: Default and Static Methods

## Learning Goals

- Learn when and how to use default methods in interfaces.
- Discover when to declare a static method in an interface.

## Introduction

We have seen interfaces so far use abstract methods. Like the `Swimmable`
interface defined a `swim()` method with no method body, forcing the
implementing classes to override the method and implement the method body.

```java
public interface Swimmable {
    void swim();
}
```

In passing, we have mentioned that an interface may also define default and
static methods. We haven't really talked much about those types of methods in
interfaces.

That is, until now.

Let's look at the "how", "when", and "why" to use default and static methods
in interfaces.

## Default Methods in Interfaces

Default methods in interfaces are also implicitly public; however, we can
declare a default method using the keyword `default`. But the biggest difference
between default methods and the abstract methods we have seen prior is that
default methods provide an implementation for the method.

Let's take a look at a `Vehicle` interface:

```java
public interface Vehicle {
    
    String getMake();
    String getModel();
    
    default String turnAlarmOn() {
        return "Turning the alarm on.";
    }
    
    default String turnAlarmOff() {
        return "Turning the alarm off.";
    }
}
```

The reason why Java introduced default methods in interfaces is to deal with the
issue of designing interfaces. As we add more abstract methods to an interface,
the more maintenance is required to implement those methods in all the
interface's implementing classes. Default methods enable developers to add new
functionality to the interfaces and ensure binary compatibility with code
written for older versions of those interfaces. This allows classes that don't
override the default methods to still work as expected while also allowing
other classes, when appropriate, to override the default methods.

Let's create an implementing class of the `Vehicle` interface. We'll create a
`Car` class:

```java
public class Car implements Vehicle {

    private String make;
    private String model;

    public Car(String make, String model) {
        this.make = make;
        this.model = model;
    }


    @Override
    public String getMake() {
        return make;
    }

    @Override
    public String getModel() {
        return model;
    }
}
```

Notice that the `Car` class does not override the default methods declared in
the `Vehicle` interface. This shows how we can have a class in a codebase and
not have it implement the default methods.

Consider the driver class below to help show us how default methods work in
interfaces:

```java
public class Main {

    public static void main(String[] args) {

        // Instantiate a Car
        Vehicle car = new Car("Jeep", "Cherokee");

        System.out.println(car.getMake());
        System.out.println(car.getModel());
        System.out.println(car.turnAlarmOn());
        System.out.println(car.turnAlarmOff());
    }
}
```

As we can see, even though the `Car` class did not override the `turnAlarmOn()`
or `turnAlarmOff()` methods, they are still available for our `car` to use. If
we added more default methods to the `Vehicle` interface, the `Car` class would
still work as is without needing to override any abstract methods.

Let's step through this code to get a better idea of what is happening. We'll
set a breakpoint at the line `System.out.println(car.turnAlarmOn());`:

![breakpoint](https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/debugger-interface-default-method-breakpoint.png)

We can see the `Car` that we have instantiated below in the debug console.

Let's use the step-into action to take us into `car.turnAlarmOn()`:

![step-into-Vehicle](https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/java-visualizer-turnalarmon.png)

The execution has jumped to the `Vehicle` interface class within the default
method for `turnAlarmOn()`! Since the `Car` class did not override the default
method, when the `car` variable calls `turnAlarmOn()`, its only way to access
that method is to go back to the `Vehicle` interface and call it there. If we
step-over this line twice, we'll see "Turning the alarm on" in the console.

### Comprehension Check

<details>
    <summary>What happens when we step into <code>car.turnAlarmOff()</code>?</summary>

  <p>Answer: <br>
     <p>The execution jumps to the <code>Vehicle</code> interface again and calls the default method for <code>turnAlarmOff()</code></p>
  </p>

  <img src="https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/java-visualizer-turnalarmoff.png" alt="turnAlarmOff">

</details>

Let's modify the `Car` class to override the `turnAlarmOn()` method:

```java
    @Override
    public String turnAlarmOn() {
        return String.format("Turning the car alarm on for %s %s", make, model);
    }
```

We'll re-run the `Main` driver class again and stop at the
`System.out.println(car.turnAlarmOn());`

<details>
    <summary>What happens when we step into the <code>car.turnAlarmOn()</code> method this time?</summary>

  <p>Answer: <br>
     <p>The execution jumps to the <code>Car</code> class and calls the <code>turnAlarmOn()</code> method there since it overrides the default method in the <code>Vehicle</code> interface.</p>
  </p>

  <img src="https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/debugger-interface-override-turnalarmon.png" alt="override-turnAlarmOn">

</details>

## Default Methods and The Diamond Problem

Consider we have another interface called `Alarm` and it has the exact same
default methods as the `Vehicle` interface:

```java
public interface Alarm {

    default String turnAlarmOn() {
        return "Turning the alarm on.";
    }

    default String turnAlarmOff() {
        return "Turning the alarm off.";
    }
}
```

In the Polymorphism lesson, we saw that it is valid to implement two different
interfaces. So let's implement the `Alarm` in our `Car` class too:

```java
public class Car implements Vehicle, Alarm {

    private String make;
    private String model;

    public Car(String make, String model) {
        this.make = make;
        this.model = model;
    }


    @Override
    public String getMake() {
        return make;
    }

    @Override
    public String getModel() {
        return model;
    }

    @Override
    public String turnAlarmOn() {
        return String.format("Turning the car alarm on for %s %s", make, model);
    }
}
```

But there is an issue now with our `Car` class. If we try to compile it, we get
the following error:

```text
java: types org.example.Vehicle and org.example.Alarm are incompatible;
  class org.example.Car inherits unrelated defaults for turnAlarmOff() from types org.example.Vehicle and org.example.Alarm
```

What we have just run into is the infamous **diamond problem**. The diamond
problem is when a class inherits two methods that are the same. In this case,
the compiler isn't sure which `turnAlarmOff()` method the `Car` should call
since it implements both the `Vehicle` interface and the `Alarm` interface.
This problem is actually why multiple inheritance is not supported in Java.
Unfortunately, it comes back to us with default methods and interfaces.

Okay, so how do we solve it?

Notice how the `turnAlarmOn()` method wasn't flagged in the compiler error.
That is because we have already overridden the `turnAlarmOn()` method. When we
run into this issue, we **must** explicitly provide an implementation for the
methods in question. In this case, we have to override the `turnAlarmOff()`
method now.

```java
package org.example;

public class Car implements Vehicle, Alarm {

    private String make;
    private String model;

    public Car(String make, String model) {
        this.make = make;
        this.model = model;
    }


    @Override
    public String getMake() {
        return make;
    }

    @Override
    public String getModel() {
        return model;
    }

    @Override
    public String turnAlarmOn() {
        return String.format("Turning the car alarm on for %s %s", make, model);
    }
    
    
    // We must override the default methods
    @Override
    public String turnAlarmOff() {
        return String.format("Turning the car alarm off for %s %s", make, model);
    }
}
```

Now when we run our `Main` driver class, it will call the overridden methods
here in the `Car` class instead of the default methods, just like how we handled
abstract methods in interfaces. The output now would look like this:

```text
Jeep
Cherokee
Turning the car alarm on for Jeep Cherokee
Turning the car alarm off for Jeep Cherokee
```

## Static Methods in Interfaces

We can also add static methods to interfaces as of Java 8. Remember, a static
method is a method that is associated with the class in which it is defined
rather than with an object.

Let's modify our `Vehicle` class a little:

```java
public interface Vehicle {

    String getMake();
    String getModel();

    default String turnAlarmOn() {
        return "Turning the alarm on.";
    }

    default String turnAlarmOff() {
        return "Turning the alarm off.";
    }

    static int getHorsePower(int rpm, int torque) {
        return (rpm * torque) / 5252;
    }
}
```

A static method can be defined in an interface the same way it can be defined in
a regular class. Notice with a static method, we also write the implementation
within the interface as well.

Static methods in interfaces were introduced to increase cohesion. Per the
Javadocs, "This makes it easier for you to organize helper methods in your
libraries; you can keep static methods specific to an interface in the same
interface rather than in a separate class"
([Oracle Javadocs: Default Methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)).

Let's go back to our `Main` driver class and get the horsepower for a `Vehicle`:

```java
public class Main {

    public static void main(String[] args) {

        // Instantiate a Car
        Vehicle car = new Car("Jeep", "Cherokee");

        System.out.println(car.getMake());
        System.out.println(car.getModel());
        System.out.println(car.turnAlarmOn());
        System.out.println(car.turnAlarmOff());
        
        // New line added to print the horsepower
        System.out.println(Vehicle.getHorsePower(2500, 480));
    }

}
```

Notice how we call the static method with the interface name preceding the
method name instead of one of its implementing classes. If we set a breakpoint
at the `System.out.println(Vehicle.getHorsePower(2500, 480));` line:

![breakpoint](https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/debugger-interface-static-method-breakpoint.png)

And we step-into the `Vehicle.getHorsePower(2500, 480);` method:

![step-into-static-method](https://curriculum-content.s3.amazonaws.com/java-mod-3/static-default-methods-interfaces/debugger-interface-stepinto-static-method.png)

We'll see that the execution jumps to the `Vehicle` interface and calls the
static method `getHorsePower()`. As we can see, it is the same control flow of
what happens when we call a static method in a class.

When we resume the program, we'll see the following output in the console:

```text
Jeep
Cherokee
Turning the car alarm on for Jeep Cherokee
Turning the car alarm off for Jeep Cherokee
228
```

## Resources

- [Baeldung: Stating and Default Methods in Interfaces in Java](https://www.baeldung.com/java-static-default-methods)
- [Oracle Javadocs: Default Methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)
- [Digital Ocean: Java 8 Interface Changes - static method, default method](https://www.digitalocean.com/community/tutorials/java-8-interface-changes-static-method-default-method)
