# Testing

There's a variety of tests one can write, which sits at different scales of depth and breadth.  

At the end, writing good tests will make our code less buggy, reduce costs, allows us to refactor with confidence,  
improves design, and can be used as documentation.

## Different test types

### Unit tests

Testing single units, such as classes and methods.  
Unit testing is all about testing the behavior of a class/method, as others would use it. It's not about testing
private members of the class.  
  
**DO** only test the public interface of the class  
**DO** test all edge cases!!  
**DO** only test one execution path at a time - this means e.g. writing two tests if a method contains branching, such as an if-statement  
**DO** mark private members/classes internal if you really want to test the behavior. Add the test project as a friend-assembly     
**DON'T** make private members public just to test them  

### Integration tests 
Testing how different parts of a system that may interact with 3rd party APIs and the like.

### Subcutaneous 
Sits just under the UI, this may for instance be testing controllers in MVC.

### UI
Test the actual application using tools that can simulate e.g. button clicks, navigation, etc.  
Keep in mind UI tests are very brittle and slow to execute. Even the smallest changes can break the test.  
These should preferably first be written when the system is in a fairly steady state.


## Writing tests
Write tests in three logical phases  
1. Arrange
    Set things up  
    Create object instances
    Create test data / inputs
2. Act
    Execute code
    Call methods
    Set properties
3. Assert
    Check results
  
### Grouping
With xUnit, you can group tests using the Trait attribute above the test class or its methods.  
 
### Shared context (Setup)
Use fixtures to share objects between test methods.
 
### Data-driven tests
Have a single test method run multiple times using different data sets.  
In xUnit, this is done using the Theory and InlineData attributes.  

## Mocking
Mocking is the replacement of production-time dependencies with test-time only dependencies in unit tests.
Testing the real dependencies should not be neglected, but should be done so in integration tests.  

## Fakes, dummies, mocks, and stubs
Fakes  
: Working implementation but not suitable for production  
: E.g. EF Core In-Memory  

Dummies  
: Passed around  
: May never used or accessed  
: Only to satisfy parameters    

Stubs  
: Provide answers to calls  
: Property gets  
: Method return values  

Mocks  
: Expect/verify calls  
: Properties  
: Methods  

## Making Code Easy to Test

Things to consider when writing code that is easily tested, are:
1. Creating seams
2. Constructing testable objects
3. Working with dependencies
4. Managing application state


### Test Driven Development
Follows a Red-Green-Refactor approach.  

Tests simplifies development because you won't have to run the whole application to see if something works.  

**Heuristics**
1. Write all requirements as tests. One test per requirement  
2. Write a test first - then the implementation   
3. Write tests for all branches. For instance two tests if there's an if-statement in a method
4. It's okay to let Fake methods return a value without doing anything
