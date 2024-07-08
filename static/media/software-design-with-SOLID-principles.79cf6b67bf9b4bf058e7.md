In the ever-evolving world of software development, the need for robust and maintainable code is more vital than ever. Writing code that stands the test of time, adapts to changing requirements, and minimizes bugs and technical debt is the goal of every developer. To achieve this, our software developers turn to a set of guiding principles that help us craft software that is both elegant and resilient.

![SOLID Principle](/images/blogs-images/solid-principles.png)
# What is SOLID?
<br>The foundation of SOLID principles was laid by Robert C. Martin in his insightful 2000 essay titled "Design Principles and Design Patterns." It's worth noting that the memorable acronym "SOLID" was coined later by Michael Feathers.
<br>In his influential essay, Martin aptly recognized that the world of software is dynamic and ever-evolving. As software evolves, it tends to grow in complexity. Without a robust design foundation, Martin cautioned that software can take on unfavourable characteristics, becoming rigid, fragile, immobile, and resistant to change. It's within this context that the SOLID principles emerged, as a response to the need for effective design patterns that address these challenging aspects of software development.

<br>The SOLID acronyms stand for:
- Single Responsibility Principle (SRP)
- Open/Close Principle (OCP)
- Liskov Substitution Principle (LSP)
- Interface Segregation Principle (ISP)
- Dependencies Inversion Principle (DIP)
<br>These principles are not just acronyms but a set of time-tested guidelines that offer invaluable insights for building software that is flexible, maintainable, and efficient.

# Single Responsibility Principle (SRP)

![Single Responsibility Principle](/images/blogs-images/single-responsibility.png)
## What is SRP?
<br>The Single Responsibility Principle is the first cornerstone of SOLID. If it's the first time you read about it, it might seem complex, but it's not. In simpler terms, it simply means that the class should only provide a focused functionality or address a particular aspect of a desired functionality hence the single responsibility. By adhering to SRP, we can avoid the pitfalls of monolithic, convoluted code.
<br>This principle has not only improved our code readability but also made maintenance a breeze. When a class has only one reason to change, it's easier to pinpoint issues, make modifications, and add new features without causing unintended side effects. SRP helps us create more robust code and less prone to bugs.

## Example
Below is an example in which we have a UserAccountController class that is responsible for handling incoming HTTP requests and returning an appropriate HTTP response for registering a user account. In this controller, we should break down our code into smaller, more manageable pieces, each with a single responsibility. We can do this by separating the responsibility of persisting the new user account into a separate class - UserAccountReposistory and validating the input data to create a new user account into another one called UserValidator. Now, our UserAccountController is easier to understand and maintain. Moreover, when we need to change how user accounts are persisted or validated, we can do so in isolation without affecting other parts of our codebase.<br>
<br>Below is an example in which we have a UserAccountController class that is responsible for handling incoming HTTP requests and returning an appropriate HTTP response for registering a user account. In this controller, we should break down our code into smaller, more manageable pieces, each with a single responsibility. We can do this by separating the responsibility of persisting the new user account into a separate class - UserAccountReposistory and validating the input data to create a new user account into another one called UserValidator. Now, our UserAccountController is easier to understand and maintain. Moreover, when we need to change how user accounts are persisted or validated, we can do so in isolation without affecting other parts of our codebase.
<br />

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
public class UserAccountController {

    private final UserAccountRepository repository;
    private final UserObjectMapper userObjectMapper;
    private final UserValidator userValidator;

    public UserController(UserAccountRepository repository, UserObjectMapper userObjectMapper, UserValidator userValidator) {
        this.userAccountRepository = userAccountRepository;
	this.userAccountMapper = userAccountMapper;
        this.userAccountValidator = userAccountValidator;
    }

    @PostMapping("/register")
    public ResponseEntity<String> registerUser(@RequestBody String userJson) {
        try {
            User userAccount = userAccountMapper.mapToUser(userJson);

            if (userAccountValidator.validateUserAccount(userAccount)) {
                userAccountRepository.save(newUserAccount);
                return new ResponseEntity<>("User registered successfully", HttpStatus.CREATED);
            } else {
                return new ResponseEntity<>("Invalid user data", HttpStatus.BAD_REQUEST);
            }
        } catch (Exception e) {
            return new ResponseEntity<>("Server Error", HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```

## Violation signs
To adhere to the Single Responsibility Principle (SRP), we should strive to have classes and methods that have a single, clear responsibility. Here are some symptoms that our code may be violating the SRP:
- **Large and Complex Classes**: If a class contains a lot of methods and instance variables, and it's difficult to describe its purpose in one sentence, it might have multiple responsibilities.
- **Frequent Code Changes**: A class that requires changes for various reasons, such as changes in requirements or different teams requesting modifications, could be a sign of multiple responsibilities.
- **High Coupling**: When changing one part of the class requires changes in multiple other parts, it may indicate that the class is responsible for more than one thing.
- **Low Cohesion**: Low cohesion occurs when a class's methods don't work closely together to achieve a common goal but instead serve various purposes.
- **Testing Challenges**: Difficulty in writing unit tests for a class due to its complexity or multiple responsibilities can be a sign of SRP violations.
- **Code Smells**: Common code smells like long methods, large switch statements, or duplicated code within a class can be indicators of SRP violations.
When we encounter these symptoms in our code, it's a good idea to consider refactoring to create classes and methods that follow the Single Responsibility Principle more closely. By doing so, whenever modifications are required, we can make changes to our code in an organized and structured fashion.<br>
<br />

# Open/Close Principle (OCP)
![Open/Close Principle](/images/blogs-images/open-close.png)
## What is OCP?
<br>The Open-Closed Principle advocates for code that is open for extension but closed for modification. Initially, this concept might seem paradoxical, but it's a powerful idea. It encourages us to design code in a way that allows for the addition of new functionality without altering the existing code. In simpler terms, we should completely avoid modifying the existing base classes and use inheritance and overriding in order to achieve an extension of existing behaviour.
<br>When it comes to code maintenance, instead of making changes to existing, stable code, we should extend it by creating new classes or modules. This approach preserves the integrity of the existing codebase and minimizes the risk of introducing bugs. Following the OCP, our code would be more adaptable and future-proof.

## Example
<br>Suppose we are developing a Spring Boot security library, and we want to allow our library users to extend the authentication mechanism without modifying our library's code.
<br><b>Define an Interface</b>: Let's start by defining an interface for an authentication provider in our library. This interface could be part of the library's public API.
<br />

```java
public interface AuthenticationProvider {
    boolean authenticate(String username, String password);
}
2. Library Implementation: In our library, we create a default implementation of this interface that suits the most common use cases.

// Closed for modification
public abstract class DefaultAuthenticationProvider implements AuthenticationProvider {
    @Override	
    public abstract boolean authenticate(String username, String password) {
        // Our authentication logic here
  }
}
3. Spring Configuration: In our library, configure Spring to use the default implementation.

@Configuration
public class SecurityConfig {
	@Bean
	public AuthenticationProvider authenticationProvider () {
		return new DefaultAuthenticationProvider();
	}
}
4. Usage by Clients: Users of our library can now use the default authentication provider.

@Autowired
private AuthenticationProvider authenticationProvider;
5. Extension: Users who want a custom authentication mechanism can create their implementation of the AuthenticationProvider interface.

public class CustomAuthenticationProvider implements AuthenticationProvider {
    @Override
    public boolean authenticate(String username, String password) {
	// Customed authentication logic here
    }
}
6. Client Configuration: Users can now configure their custom implementation in their Spring Boot application:

@Configuration 
public class CustomSecurityConfig {
    @Bean   
    public AuthenticationProvider authenticationProvider() {   
        return new CustomAuthenticationProvider();  
    }  
```

By following this approach, our library remains closed for modification. We can extend it by implementing the provided interface and configuring our own implementations. This adheres to the Open/Closed Principle, as we allow for extension without altering the existing codebase. 

## Violation signs
Signs that indicate a violation of the Open/Closed Principle are:
- **Modifying Existing Code**: If you frequently need to modify the existing code to add new features or make changes, it suggests that the code is not closed for modification. This is a clear violation of the OCP.
- **Large Switch or If-Else Statements**: When you see large switch or if-else statements that handle different cases or behaviours, it's an indication that the code is not open for extension. Adding a new case or behaviour requires modifying the existing code, violating the principle.
- **Subclasses for Every New Feature**: If you find yourself creating new subclasses for every new feature or behaviour, it's a sign that you're not extending existing code but rather modifying it to accommodate the changes.
- **Monolithic Classes**: Classes that are large and have multiple responsibilities are likely violating the OCP. Smaller, focused classes are more likely to adhere to the principle because they can be extended through inheritance or composition without modifying the existing code.
- **Tight Coupling**: When changing one part of the code forces changes in many other places, it indicates that the code is not closed for modification. Tight coupling between components is a violation of the OCP.
- **Inadequate Abstraction**: If you have to make changes at a low level in the code to add new functionality, it's an indication of a violation. Adequate abstraction should allow you to extend the software without touching existing implementations.
- **Lack of Interfaces or Abstract Classes**: The absence of interfaces or abstract classes that define clear extension points in your code can indicate a violation. These constructs are essential for adhering to the OCP by providing well-defined points for extension.
<br>To adhere to the Open/Closed Principle, it's important to structure our code in a way that allows us to add new functionality without modifying existing code. This is typically achieved through techniques like abstraction, inheritance, and interfaces, which provide clear extension points for our software.

# Liskov Substitution Principle (LSP)
![Liskov Substitution Principle](/images/blogs-images/liskov-substitution.png)
## What is LSP?
The Liskov Substitution Principle is named after the computer scientist Barbara Liskov. It defines a set of guidelines that ensure the smooth interchangeability of objects of a derived class with objects of its base class without affecting the correctness of the program.
<br>In simpler terms, if a program is using a base class, it should be able to use any of its derived classes without knowing the difference. This principle emphasizes polymorphism and encapsulation, which are core concepts in Object Oriented Programming (OOP).
<br>By adhering to LSP, we can create code that is more predictable and easier to maintain. When new subclasses seamlessly integrate into the existing codebase, it reduces the likelihood of introducing unforeseen issues during maintenance or future enhancements. And LSP can help developers write code that is reliable and less error-prone.

##Example
Suppose we are building an e-commerce application, and we have a PaymentProcessor class that handles various payment methods. We want to extend this class to handle a new payment method - StripePayment. To adhere to LSP, the derived class should be a valid subtype of the base class, which means it should be able to replace the base class without breaking the application.
<br>We have a base class called BasePaymentProcessor that provides a method called processPayment. The base class contains common payment processing logic.

```java
interface PaymentProcessor {
    public void processPayment(double amount);
}

// Base class
public class BasePaymentProcessor {
    public void processPayment(double amount) {
        // Common payment processing logic       
    }
}
```

2. Next, we create a derived class StripePaymentProcessor that extends the base class. This derived class overrides the processPayment method to include additional logic specific to Stripe payments.<br>

```java
// Derived class
public class StripePaymentProcessor extends BasePaymentProcessor {
    @Override
    public void processPayment(double amount) {
        // Additional logic for Stripe payments
    }
}
```

3. In our PaymentService class, we inject an instance of PaymentProcessor and call the processPayment method to handle a payment.<br>

```java
@Service
public class PaymentService {
    @Autowired
    public void processPayment(PaymentProcessor paymentProcessor, double amount) {
        paymentProcessor.processPayment(amount);
    }
}
```

4. Now, we can pass either the base class or a derived class (such as StripePaymentProcessor) to this method without any issues. The application remains unaware of which specific payment method is being used.<br>

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Autowired
    private PaymentService paymentService;

    public void processStripePayment() {
        PaymentProcessor stripePaymentProcessor = new StripePaymentProcessor();
        paymentService.processPayment(stripePaymentProcessor, 100.0);
    }
}
```

## Violation Signs
Violations of the Liskov Substitution Principle can manifest through various signs, including:
- **Different Behavior in Subclasses**: Subclasses override or extend the behaviour of a parent class in a way that is not compatible with the parent class's behaviour. This means that the child class behaves differently from what the clients of the parent class expect.
- **Overriding Methods with Stronger Preconditions/Weaker Postconditions**: A subclass tightens the preconditions/weakens the postconditions of a method when compared to the parent class. This can lead to situations where the subclass expects stricter conditions for method invocation or it may not fulfill the same guarantees after method execution as the parent class.
- **Throwing More Specific Exceptions**: A subclass throws exceptions that are more specific than the exceptions thrown by the parent class. This can lead to unexpected exceptions for clients that only expect the parent class's exceptions.
- **Inability to Replace Base Class**: If you find that you cannot use a subclass to replace the parent class in the code without causing unexpected behaviour or runtime errors, it's a sign that the Liskov Substitution Principle is being violated.
- **Loss of Expected Invariants**: When using a subclass, the invariants (properties or conditions that should always be true) that hold with the parent class are no longer guaranteed. Subclasses should preserve the invariants of the parent class.
- **Unexpected Side Effects**: The presence of unexpected side effects when using a subclass is a sign of a violation. Subclasses should not introduce new side effects that were not present in the parent class.
- **Conditional Checks**: Clients of the class need to introduce conditional checks or type checks to handle different behaviours of subclasses. This goes against the idea that clients should be able to use subclasses interchangeably.
- **Interfaces and Contracts**: Violations can also occur when classes implementing interfaces don't honour the contracts implied by those interfaces. If a subclass doesn't fulfil the contract of an interface, it violates LSP.
In essence, the Liskov Substitution Principle emphasizes that derived classes should extend or specialize the behaviour of the base class in a way that is consistent with the expected behaviour of the base class. Violations of LSP can lead to code that is difficult to understand, maintain, and predict, as well as causing unexpected runtime issues.

# Interface Segregation Principle (ISP)
![Interface Segregation Principle](/images/blogs-images/interface-segregation.png)
## What is ISP?
The Interface Segregation Principle (ISP) focuses on the relationships between interfaces and the classes that implement them. It promotes the creation of smaller, more focused interfaces rather than large, monolithic ones.
<br>In simple terms, we should not try to put unrelated methods in one big interface and make all other classes in our system implement it.
<br>By following ISP, we can reduce the chances of classes being forced to implement methods they don't need. This leads to cleaner and more maintainable code because classes are not burdened with unnecessary responsibilities. ISP encourages a more modular and cohesive design, making it easier to adapt and maintain code over time.

## Example
Let's consider an example where we have an application that manages different types of vehicles, such as cars and bicycles. We want to adhere to the Interface Segregation Principle by creating smaller, more focused interfaces.
<br>Define Interfaces: First, let's create a separate interface for the common behaviours of cars and bicycles, instead of a single large interface, and interfaces that extend this common Vehicle interface with their own behaviours: MotorizedVehicle and NonMotorizedVehicle

```java
public interface Vehicle {
    void start();
    void stop();
}

public interface MotorizedVehicle extends Vehicle {
    void accelerate();
    void brake();
}

public interface NonMotorizedVehicle extends Vehicle {
    void pedal();
}
```

2. Implement Classes: Create classes for different types of vehicles, implementing the appropriate interfaces.<br>

```java
public class Car implements MotorizedVehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }

    @Override
    public void stop() {
        System.out.println("Car stopped");
    }

    @Override
    public void accelerate() {
        System.out.println("Car accelerated");
    }

    @Override
    public void brake() {
        System.out.println("Car braked");
    }
}

public class Bicycle implements NonMotorizedVehicle {
    @Override
    public void start() {
        System.out.println("Bicycle started");
    }

    @Override
    public void stop() {
        System.out.println("Bicycle stopped");
    }

    @Override
    public void pedal() {
        System.out.println("Bicycle pedaled");
    }
}
```

3. Usage: In our application, we can use these interfaces and classes to manage different types of vehicles. Each class implements only the methods relevant to its type.<br>

```java
public class VehicleManager {
    public void operateVehicle(Vehicle vehicle) {
        vehicle.start();
        // Perform some operations
        vehicle.stop();
    }
}
```

By adhering to the Interface Segregation Principle, we've designed a more flexible system. We have smaller, focused interfaces that allow each class to implement only the methods it needs. This way, we avoid forcing unnecessary methods on classes that don't require them, improving maintainability and readability in the long run.

## Violation Signs
Violations of the Interface Segregation Principle (ISP) can lead to design issues in our code. Here are some signs of violations:
- **Large Interfaces**: An interface contains many methods, some of which are not relevant to the implementing classes. This can force classes to implement methods they don't need, leading to bloated and confusing code.
- **Forced Empty Implementations**: Implementing classes are forced to provide empty implementations (no-op) for methods they don't use, which adds unnecessary boilerplate code.
- **Client Classes Needing Irrelevant Methods**: Client classes (classes that implement the interface) are required to use methods that they don't need or care about.
- **Tight Coupling**: The use of a large, monolithic interface can lead to tight coupling between classes, reducing the flexibility of the code.
- **Maintenance Challenges**: Modifying a large interface can have a ripple effect on many classes, making maintenance more complex.
- **Violation of Single Responsibility Principle**: A class that implements a large interface may have multiple responsibilities, violating the SRP.
To adhere to the Interface Segregation Principle, we should design interfaces that are cohesive and tailored to the specific needs of implementing classes. This results in a more maintainable and flexible codebase.

# Dependency Inversion Principle (DIP)
![Dependency Inversion Principle](/images/blogs-images/dependencies-inversion.png)
## What is DIP?
The Dependency Inversion Principle (DIP) focuses on decoupling high-level modules from low-level modules by depending on abstractions, not concrete implementations.
<br>This principle consists of two key guidelines:
- High-level modules should not depend on low-level modules. Both should depend on abstractions.
- Abstractions should not depend on details. Details should depend on abstractions.
- In simpler terms, the DIP encourages the use of abstractions, such as interfaces or abstract classes, to define the interactions between components or modules within a system. These abstractions serve as contracts or protocols that high-level and low-level modules adhere to.

## Example
Let's consider a simple application that sends notifications to users through different channels, such as email and SMS. We'll apply the DIP to ensure that high-level modules don't depend on low-level modules and both depend on abstractions.
<br>Create an Interface: Let's create an interface for the NotificationService

```java
public interface NotificationService {
    void sendNotification(String message, String recipient);
}
```

2. Implement Low-Level Classes: Implement classes for sending notifications through specific channels, like Email and SMS.<br>

```java
@Component
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // Implementation to send an email notification
    }
}
@Component
public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // Implementation to send an SMS notification
    }
}
```

3. Create a High-Level Class: Create a high-level class, such as a NotificationManager, that uses the NotificationService interface for sending notifications. The NotificationManager class doesn't depend on specific implementations but relies on the abstraction.<br>

```java
@Service
public class NotificationManager {
    private final NotificationService notificationService;

    @Autowired
    public NotificationManager(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void sendNotification(String message, String recipient) {
        // High-level logic
        notificationService.sendNotification(message, recipient);
    }
}
```

4. Inject Specific Notification Service Beans: When we need to use a specific notification service, we can inject it into the NotificationManager via the constructor.<br>

```java
@Service
public class NotificationServiceController {

private final NotificationManager notificationManager;
    @Autowired
    public NotificationServiceController(@Qualifier("emailNotificationService") NotificationService emailNotificationService) {
        this.notificationManager = new NotificationManager(emailNotificationService);
    }
}
```

With this approach, we can easily switch between different notification services by injecting the desired implementation into the NotificationManager. Spring will manage the dependencies and resolve them at runtime based on our configuration.

# Conclusion
In conclusion, the SOLID principles provide a set of fundamental guidelines for writing clean, maintainable, and flexible software. By adhering to these principles, software developers can create code that is easier to understand, extend, and modify.
<br>Throughout this blog post, we've explored each of the five SOLID principles: the Single Responsibility Principle, the Open/Closed Principle, the Liskov Substitution Principle, the Interface Segregation Principle, and the Dependency Inversion Principle. We've seen how they help address common issues in software development, such as code complexity, brittleness, and tight coupling, and how they encourage the use of design patterns and best practices.

<br>By applying the SOLID principles in our software development projects, we can:
- Create code that is easier to maintain and extend.
- Encapsulate behaviour and responsibilities effectively.
- Reduce code duplication and increase reusability.
- Minimize the impact of changes in one part of the codebase on other parts.
- Promote a better separation of concerns, leading to more organized and modular code.
- These principles are not rigid rules but rather a set of guiding philosophies that help us make better design decisions. Applying them appropriately depends on the context and requirements of our specific project. Ultimately, mastering the SOLID principles takes time and practice, but the effort is well worth it in terms of software quality, developer productivity, and long-term maintainability.

As we continue our software development journey, let's keep the SOLID principles in mind and strive to apply them in our projects. By doing so, we'll be on our way to becoming more skilled and effective as a software engineer.