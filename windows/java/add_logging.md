## Sample Code for logging PetClinic app requests

Pet Clinic does not generate any application logs for user requests. So we add some code to generate log entries.

Open the ***.\src\main\java\org\springframework\samples\petclinic\owner\OwnerController.java*** file.
```
notepad .\src\main\java\org\springframework\samples\petclinic\owner\OwnerController.java
```

Add this import statement (import statements are usually found at the top of the java source file)
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

Add a static logger field to the **OwnerController** class
```java
private static final Logger logger = LoggerFactory.getLogger(OwnerController.class);
```

Find the following one or more log entries in the **showOwner** method.
```java
@GetMapping("/owners/{ownerId}")
public ModelAndView showOwner(@PathVariable("ownerId") int ownerId) {
    ModelAndView mav = new ModelAndView("owners/ownerDetails");
    Optional<Owner> optionalOwner = this.owners.findById(ownerId);
    Owner owner = optionalOwner.orElseThrow(() -> new IllegalArgumentException(
            "Owner not found with id: " + ownerId + ". Please ensure the ID is correct "));
    mav.addObject(owner);
    
    //add log statement
    logger.info("Owner record accessed for {} {}", owner.getFirstName(), owner.getLastName());
    
    return mav;
}
```

Run java format to fix any formatting
```cmd
.\mvnw spring-javaformat:apply
```

Rebuild package
```cmd
.\mvnw package -Dmaven.test.skip=true
```

