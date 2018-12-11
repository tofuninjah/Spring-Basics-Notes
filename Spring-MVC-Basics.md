## Spring MVC Basics

### Configuring Spring Beans

	// applicationContext.xml
	<beans...>
		<bean id="myCoach" class="com.luv2code.springdemo.BaseballCoach">
		</bean>
	</beans>

### Create Spring Container (Pass in name of Configuration file)

	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");


### Retrieve Bean from Container

	Coach theCoach = context.getBean("myCoach", Coach.class);

* myCoach is the Bean ID in the applicationContext
* Coach.class is the actual Interface that baseball coach implements
* BaseballCoach is the actual Implementation

### Constructor Injection

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

* Inject myFortuneService into BaseballCoach.
* Spring will look at BaseballCoach and call it's constructor and pass in a reference to myFortuneService.

### Java Annotations

	<beans>
		<context:component-scan base-package="com.multibrand" />
	</beans>

Spring will scan all Classes within the base-package, recursively and automatically register them into the Spring Container.


### Constructor Injection

    private FortuneService fortuneService;
    @Autowired
    public TennisCoach(FortuneService theFortuneService) {
    	fortuneService = theFortuneService;
    }

### Setter Injection (@autowired Annotation on a setter method)

	private FortuneService fortuneService;
	public TennisCoach() {} // Default constructor, not required
	// Create setter method for injections
	@Autowired // Any class that implements FortuneService
	public voide setFortuneService(FortuneService theFortuneService) {
	    fortuneService = theFortuneService;
	}

### Method Injection

	@Autowired // Sprint will do an injection here, on any method name!
	public void doSomeCrazyStuff(FortuneService fortuneService) {
		this.fortuneService = fortuneService;    
	}

### Field Injection

	public class TennisCoach implements Coach {
	    @Autowired // Directly on the field!!!!!!
	    private FortuneService fortuneService;
	    public TennisCoach() {
	    }
	    // No need for setter methdos ...
	}

### Annotation Autowiring and Qualifiers
	* Fortune Service
		- HappyFortuneService
		- RandomFortuneService
		- DatabaseFortuneService
		- RESTFortuneService	
	* NoUniqueBeanDefinitionException
		- No qualifying bean of type FortuneService is defined: expected single matching bean but found 4: HappyFortuneService, RandomFortuneService, DatabaseFortuneService, RESTFortuneService

**We will need to tell Spring which bean to use... with @Qualifier**

    @Component
    public class TennisCoach implements Coach {
    	@Autowired
    	@Qualifier("happyFortuneService") // Bean id, or component
    	private FortuneService fortuneService;
    	...
    }

**@Qualifier can be used with all 3 (Constructor, Setter, Field) Injection types (Only Constructors need to have it inside the parenthesis!)**

	public TennisCoach(@Qualifier("randomFortuneService") FortuneService theFortuneService) {}

### Injecting Values from  properties file using Java annotations

	// src/sport.properties
	foo.email=email@myemail.com
	foo.team=Silly Java Coders
	
	// applicationContext.xml
	// just after the <context:component-scan .../>
	<context:property-placeholder location="claspath:sport.properties"/>
	
	// SwimCoach.java
	@Value("${foo.email}")
	private String email;
	@Value("{foo.team}")
	private String team;
