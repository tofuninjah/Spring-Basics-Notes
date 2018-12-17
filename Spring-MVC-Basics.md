## Spring MVC Basics

### Configuring Spring Beans

```xml
// applicationContext.xml
<beans...>
	<bean id="myCoach" class="com.luv2code.springdemo.BaseballCoach">
	</bean>
</beans>
```

### Create Spring Container (Pass in name of Configuration file)

```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

### Retrieve Bean from Container

```java
Coach theCoach = context.getBean("myCoach", Coach.class);
```

* myCoach is the Bean ID in the applicationContext
* Coach.class is the actual Interface that baseball coach implements
* BaseballCoach is the actual Implementation

### Constructor Injection

```java
// FortuneService.java
public interface FortuenService {
   	public String getFortune();
}

// HappyFortuneService.java
public class HappyFortuneService implements FortuneService {
    public String getFortune() {
        return "Today is your lucky day!";
    }
}

// Injection
public class BaseballCoach implements Coach {
  	private FortuneService fortuneService;

    public BaseballCoach(FortuneService theFortuneService) {
    	fortuneService = theFortuneService;
    }
}

// Configurations
<bean id="myFortuneService" class="com.luv2code.springdemo.HappyFortuneService"></bean>
<bean id="myCoach" class="com.luv2code.springdemo.BaseballCoach">
    	<constructor-arg ref="myFortuneService" />
</bean>
```

* Inject myFortuneService into BaseballCoach.
* Spring will look at BaseballCoach and call it's constructor and pass in a reference to myFortuneService.

### Java Annotations

```xml
<beans>
	<context:component-scan base-package="com.multibrand" />
</beans>
```

Spring will scan all Classes within the base-package, recursively and automatically register them into the Spring Container.


### Constructor Injection

```java
private FortuneService fortuneService;

@Autowired
public TennisCoach(FortuneService theFortuneService) {
    	fortuneService = theFortuneService;
}
```

### Setter Injection (@autowired Annotation on a setter method)

```java
private FortuneService fortuneService;

public TennisCoach() {} // Default constructor, not required
	// Create setter method for injections
	
	@Autowired // Any class that implements FortuneService
	public voide setFortuneService(FortuneService theFortuneService) {
	    fortuneService = theFortuneService;
}
```

### Method Injection

```java
@Autowired // Sprint will do an injection here, on any method name!
public void doSomeCrazyStuff(FortuneService fortuneService) {
	this.fortuneService = fortuneService;    
}
```

### Field Injection

```java
public class TennisCoach implements Coach {
    @Autowired // Directly on the field!!!!!!
    private FortuneService fortuneService;
    public TennisCoach() {
    }
	// No need for setter methdos ...
}
```

### Annotation Autowiring and Qualifiers
* Fortune Service
    - HappyFortuneService
    - RandomFortuneService
    - DatabaseFortuneService
    - RESTFortuneService	
* NoUniqueBeanDefinitionException
	- No qualifying bean of type FortuneService is defined: expected single matching bean but found 4: HappyFortuneService, RandomFortuneService, DatabaseFortuneService, RESTFortuneService

**We will need to tell Spring which bean to use... with @Qualifier**

```java
@Component
public class TennisCoach implements Coach {
	@Autowired
	@Qualifier("happyFortuneService") // Bean id, or component
	private FortuneService fortuneService;
	...
}
```

**@Qualifier can be used with all 3 (Constructor, Setter, Field) Injection types (Only Constructors need to have it inside the parenthesis!)**

```java
public TennisCoach(@Qualifier("randomFortuneService") FortuneService theFortuneService) {}
```

### Injecting Values from  properties file using Java annotations

```java
// src/sport.properties
foo.email=email@myemail.com
foo.team=Silly Java Coders

// SportConfig
@Configuration
@PropertySource("classpath:sport.properties") // Loads properties
public class sprtConfig {
	...
}

// applicationContext.xml
// just after the <context:component-scan .../>
<context:property-placeholder location="claspath:sport.properties"/	

// SwimCoach.java
@Value("${foo.email}")
private String email;
@Value("{foo.team}")
private String team;
```

### Bean Lifecycle with Annotations

* Custom code during **bean initialization** or **bean destruction** 
* Custom business logic methods
* Setting up handles to resources (db, sockets, file, etc...)

1. Define your methods for *init* and *destroy*
2. Add annotations: *@PostConstruct* and *@PreDestroy*
3. void is the most common return type
4. Cannot accept any arguments

Configuration:

```java
@Component
public class TennisCoach implements Coach {
	@PostConstruct
	public void doMyStartupStuff() { ... }
	
	@PreDestroy
	public void doMyCleanupStuff() { ... }
}
```

### Pure Java Configuration

To review: 1. Full XML Config, 2. XML Component Scan, 3. Java Configuration Class (No XML!)

1. Create Java class and annotate as @Configuration
2. Add component scanning support: @ComponentScan (optional)
3. Read Spring Java configuration class
4. Retrieve bean from Spring container

Example:

```java
// Step 1 -> Add @Configuration to SportConfig
@Configuration
public class SportConfig {
	...
}

// Step 2 -> Add @ComponentScan to SportConfig
@Configuration
@ComponentScan("com.packange.name")
public class SportConfig {
   	...
}

// Step 3 -> Read Spring Java configuration
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SportConfig.class);

// Step 4 -> Retrieve bean from Spring Container
Coach theCoach = context.getBean("tennisCoach", Coach.class);
```

### Define Spring Beans with Java Code

1. Define method to expose bean
2. Inject bean dependencies
3. Read Spring java configuration class
4. Retrieve bean from Spring container
	

Example:

```java
// Step 1 -> Define method to expose bean
@Configuration
public class SportConfig {
    @Bean
    public Coach swimCoach() { // Bean Id
    	SwimCoach mySwimCoach = new SwimCoach();
    	return mySwimCoach;
    }
}

// Step 2 -> Inject the bean dependencies
@Configuration
public class SportConfig {
    @Bean
    public FortuneService happyFortuneService() { // bean id
   		return new HappyFortuneService();
    }
    
    @Bean
    public Coach swimCoach() { // Bean Id
        return new SwimCoach( happyFortuneService() );
    }
}

// Step 3 -> Read Spring Java configuration class
AnnotationConfigApplicationContext context = new AnnotationConfigAplicationContext(SprtConfig.class);

// Step 4 -> Retrieve bean form Spring Container
Coach theCoach = context.getBean("swimCoach", Coach.class);
```

**The above, we defined two Beans and injected the dependencies**

### Preprocess all Web Requests

```java
@InitBinder
public void initBinder(WebDataBinder) dataBinder) {
	StringTrimmerEditor stringTrimmerEditor = new StringTrimmerEditor(true);
	dataBinder.registerCustomEditor(String.class, stringTrimmerEditor);
}
```

// StringTrimmerEditor(true) -> Removes whitespace, TRUE means empty returns null
// Use dataBinder to register the custom editor -> for every String class, apply the stringTrimmerEditor

### Using RegEx

```java
@Pattern(regexp="^[a-zA-Z0-9]{5}", message="only 5 chars/digits")
private String postalCode;
```

### Custom Error Messages

**To remedy this error: Failed to convert property value of type java.lang.String to required type java.lang.Integer for property freePasses; nested exception is java.lang.NumberFormatException: For input string: "asdfsdf" **

```java
// src/resources/messages.properties
typeMismatch.student.freePasses=Invalid number
NotNull.student.postalCode=No Null ZipCodes!!!

servlet.xml
<!--  Load custom message resources -->
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basenames" value="resources/messages" />
</bean>
```

BindingResult has errors, and the different Error Types that need to go into messages.properties to overwrite and give custom error messages.  Ex: `System.out.println("Binding result: " + theBindingResult);`.

`[NotNull.student.firstName,NotNull.firstName,NotNull.java.lang.String,NotNull]`

Summary: println the BindingResult, and from there you can create a custom error message based on what type of error was thrown.

### Custom Validation

```java
//Usage java annotations
@CourseCode(value="LUV", message="mest start with LUV")
private String courseCode;
```

1. Create custom validation rule
	a. Create @CourseCode annotation
	b. Create CourseCodeConstraintValidator (Helper class contains custom business logic for validation)

###### Step 1a: Create the @CourseCode annotation

```java
@Constraint(validatedBy = CourseCodeConstraintValidator.class)
@Target( {ElementType.METHOD, ElementType.FIELD} )
@Retention(RetentionPolicy.RUNTIME)
public @interface CourseCode {
    // Define default course code
    public String value() default "LUV";
    // Define default error message
    public String message() default "must start with LUV";
    // Define default groups
    public Class<?>[] groups() default {};    
    // Define default payloads
    public Class<? extends Payload>[] payload() default {};
}
```

* @Constraint: ValidateBy, and give it the Business Rules class
* @Target: Where can you use this annotation?  Method, OR any field.
* @Retention: How long should we retain it?  Keep it during run-time.

###### Step 1b: ConstraintValidator, actual guy that has the actual business rules.

CourseCodeConstraintValidator.java

```java
public class CourseCodeConstraintValidator implements ConstraintValidator<CourseCode, String> {
    private String coursePrefix;
    @Override
    pubic void initializeCourseCode theCourseCode) {
        coursePrefix = theCourseCode.value();
    }
    @Override
    public boolean isValid(String theCode, ConstraintValidatorContext theConstraintValidatorContext) {
        boolean result;
        if(theCode != null) {
            result = theCode.startsWith(coursePrefix);
        } else {
            result = true;
        }
        return result;
    }
}
```

Student.java

```java
import com.plainspringmvc.demo.validation.CourseCode;
@CourseCode 
private String courseCode;
```

### Hibernate Annotations

**Java class is camelCase, while Database table is _underscored_**
Step 1 => Map class to Database table
Step 2 => Map fields to database columns

```java
package com.hibernating.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="student")
public class Student {
	
	@Id
	@Column(name="id")
	private int id;
	
	@Column(name="first_name")
	private String firstName;
	
	@Column(name="last_name")
	private String lastName;
	
	@Column(name="email")
	private String email;
	
	public Student() {}

	public Student(String firstName, String lastName, String email) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	@Override
	public String toString() {
		return "Student [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email + "]";
	}

}
```

### Creating and saving Objects

**Two Key Players**
1. SessionFactory
	- Reads the hibernate config file
	- Creates Session objects
	- Heavy-weight object
	- Only create once in your app
2. Session
	- Wrapper around  a JDBC connection 
	- Short-lived
	- Main object used to save/retrieve objects
	- Retrieved from SessionFactory

```java
public static void main(String[] args) {
    SessionFactory factory = new Configuration()
    	.configure("hibernate.cfg.xml")
    	.addAnnotatedClass(Student.class)
    	.buildSessionFactory();
    	
    Session session = factory.getCurrentSession();
    
    try  {
        // Now use the session object to save/retrieve Java objects
        
        // Create a student object
        Student tempStudent = new Student("Paul", "Wall", "paul@luv2code.com");
        
        // Start transaction
        session.beginTransaction();
        
        // Save the student
        session.save(tempStudent);
        
        // Commit the transaction
        session.getTransaction().commit();
        
    } finally {
        factory.close();
    }
}
```

### Hibernate Identity

Options
	- GenerationType.AUTO
	- GenerationType.IDENTITY
	- GenerationType.SEQUENCE
	- GenerationType.TABLE

```java
@GeneratedValue(strategy=GenerationType.IDENTITY)
```









