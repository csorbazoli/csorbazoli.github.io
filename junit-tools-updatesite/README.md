# JUnit Tools for Spring
[Update Site URL](https://csorbazoli.github.io/junit-tools-updatesite/)

## Aim of this project
Create a tool for helping the developers to write simple and maintainable unit tests with minimal effort on the tedious part and let them focus on the important part of the tests.

This plugin promotes the "Test First" approach. Create a well structured test method skeleton from the method signature with

* given-when-then blocks (Gherkin style)
    - given: pre-initialized parameter values
    - when: method is called
    - then: assert for return value

Additional features:
* using mockito runner for fast test execution
* mock dependencies of the tested class
* using *TestValueFactory* for effortless but consistent input value creation, see samples below
* using JSON comparison for checking complex result types instead of endless asserts on each single field value
* customization through preferences

Spring support:
* use Spring testing annotations if that is preferred or needed
* enhanced rest endpoint testing with MockMvc

## Important note / disclaimer

This plugin has reasonable default settings for creating test method skeletons, but you'll need to extend and fine tune those settings based on the ever changing
requirements of your specific project. The plugin has a configurable set of default values for the input parameters, imports, naming conventions and so on. Feel free
adjust these settings according to your needs.
Please remember, that testing is a continuous effort that is part of the development task. The test implementation evolves together with the production code, 
just like the tools that we use.


## Samples

You can find the samples below on [GitHub](https://github.com/csorbazoli/junit-tools/tree/master/junit-tools-demo).
Feel free to clone this repository and play around with the test generation functionality of the plug-in.

### Testing static methods
This is the simplest case as the test class does not need any specific annotation, and no need to initialize the tested class either.

| Base class  | Test class |
| ------------- | ------------- |
| ![Static method in base class](images/sample_static_method_base.png)  | ![Test for static method](images/sample_static_method_test.png)  |

> Note, that the gherkin style comments are separating the test method body to distinct blocks that clearly shows what are prerequisites, what is tested and
what do we check.

### Testing a method with simple input parameters / return value
For simple input parameters (i.e. primitive types) the plug-in is setting some predefined default values instead of leaving them uninitialzed.
Same goes for the return value, that an assertEquals will be generated with a predefined default value.

For example

| Base class  | Test class |
| ------------- | ------------- |
| ![Method with primitive parameters](images/sample_primitive_params_base.png)  | ![Test initializing primitive parameters](images/sample_primitive_params_test.png)  |

> There will be a preference page for the plug-in to set up the default values you prefer, although it's just a formality as you'll need to customize the values anyway
according to the logic of the method you're testing.
> The main goal of this functionality to reduce the manual effort on writing the initialization steps and the assert clause.

### Testing a method without return value
When the method has no return value (i.e. void), then you'll need to test for some "side effect", like these
* the method may change one of the input parameters

```java
assertThat(param.getChangedField()).isEqualTo("new value");
```
* the method may call another service with certain arguments

```java
verify(otherService).otherMethod("some value");
```
* the method may throw an exception

```java
assertThrows(SomeException.class, () -> underTest.someMethod("some value");
```
* the method may not throw any exception

```java
assertDoesNotThrow(() -> underTest.someMethod("some value");
```

Since it's vary what the side effects can be, the plug-in only generates a `TODO` comment only.
For example

| Base class  | Test class |
| ------------- | ------------- |
| ![Method without return value](images/sample_void_method_base.png)  | ![Test for method without return value](images/sample_void_method_test.png)  |

### Testing a method with complex input parameters
One of the most tedious part of writing a unit test is the preparation of input parameters when mocking is not sufficient.
Mocking an input parameter is only useful when you have an interface or a class that is not a Java Bean (i.e. a service or function).

On the other hand, when you need to deal with data then you need to prepare that item with all it's fields that are relevant for that method.
This can require a big effort and lot of extra work.

See the example in the case above, where the method had the `DemoObject` input parameter. The generated test initializing the parameter with the
TestValueFactory.

 ![Test for method without return value](images/sample_complex_param_test.png)

Using *TestValueFactory* is a nice workaround for this problem. It prefills all fields of given data object (e.g. Java Bean) with consistent and predictable values.
So you can rely on that the input data is always the same and you only need to take care of the fields that are special related to that function. For example
* fields that should be empty/null
* text fields with some special formatting expectation (e.g. date/time values)

But in most of the cases it's enough to have the default values generated by the factory.

If the input parameter is not a primitive value, then the generator assumes that the TestValueFactory should be used instead and generates to code accordingly (see below).

### Testing a method with complex return value
When the tested method is producing a complex data structure in the response, then it can be tedious to test all of details of that object.
Either you're expecting that only a handful of properties were changing as a "side-effect" of the method execution, 
or all of the fields are important because it the method is creating a new object based upon the input parameters (e.g. a converter from an entity to a DTO).

| Base class  | Test class |
| ------------- | ------------- |
| ![Method with complex return value](images/sample_complex_return_base.png)  | ![Test for method with complex return value](images/sample_complex_return_test.png)  |

In my opinion, this makes the test maintenance way easier and this way the test can detect (and protect from) accidental changes resulting from changes in the data model.
It may sound controversial, but actually this can help a lot.

For example, the data model is changing in the application and the developer would forgot that this change
affects a certain service or converter that is consuming or producing the changed element.
The test would show automatically that the output is different from the previously known and expected form.
This at least raises the important question: does this class need to be changed also in order to handle the new field, or the field removed is still needed, etc.
The answer might be "yes it's okay", then update the JSON file in the test-resources folder and done.
But in several cases the answer is "oops, I forgot about that this needs to be handled here as well".

Also note, that the usual testing that is focusing only on the known fields would not detect this problem.

The same solution can be used to check if a dependent service was triggered with the right object.
For example, if the method supposed to save an entity to the database with specific field values, then use the following pattern.

```java
	ArgumentCaptor<SomeEntity> entityCaptor = ArgumentCaptor.forClass(SomeEntity.class);
	verify(someRepo).save(entityCaptor.capture());
	assertThat(TestUtils.objectToJson(entityCaptor.getValue()))
		.isEqualTo(TestUtils.readTestFile("dbtestfiles/SomeEntity_saved.json"));
```

### Testing Spring related classes
Spring tests are slightly different than the usual basic or mocked test classes.
These tests are instantiating a Spring context when they are running, that comes with it's pros and cons.
Advantages:
* it's more comprehensive and reliable for testing Spring feature (e.g. caching)
* it can use the environment configuration properties (i.e. entries from application.yml)
* easier to write integrated tests (i.e. injecting an actual dependency)

Disadvantages:
* the overhead of initializing the Spring context can make the test execution slower
* bit more tedious to write the test (more annotations and settings)

Formal differences:
* using different annotations for the test elements (e.g. @Autowired instead of @InjectMocks, @MockBean instead of @Mmock) 

Good practices:
* Use simple mocking instead if no Spring specific functionality is tested
* Minimize the Spring context to the bare minimum
    - Mainly for the tested class only instead of building up the full application
    - Avoid initializing the database engine, unless you specifically want to test that
* Use Spring test for the following areas
    - Caching (e.g. @Cacheable, @EvictCache), that is especially useful to make sure the cache configuration works properly as it's easy to misconfigure/break
    - (REST) Controller endpoints (e.g. @GetMapping, @PostMapping), where easy to misconfigure the parameters, default values, headings, etc.
    - Database, in case you are writing custom database handling methods instead of generated ones

#### Example: Testing a Spring service
One important part of testing a class that has some dependencies (i.e. injected fields), that these fields need to be mocked properly.

You can use either Mockito or Spring extensions to execute these tests. There are slight differences in how the class and the fields are annotated, as described above.

| Base class  | Test class with MockitoExtension | Test class with SpringExtension |
| ------------- | ------------- | ------------- |
| ![Test class with dependencies](images/sample_spring_service_base.png)  | ![Test dependencies with Mockito](images/sample_spring_service_mockito_test.png)  | ![Test dependencies with Spring](images/sample_spring_service_integrated_test.png)  |

> Note, that `@ExtendWith(SpringExtension.class)` is superfluous, that's why it's not generated.

#### Example: Testing a Spring controller with different endpoints
A special case of testing Spring components when we need to write tests for REST controllers or other public endpoints of the Spring application.

The plug-in generates an enhanced test method for these endpoints that were implemented in methods annotated with some `RequestMapping` annotation (e.g. `GetMapping`).
The test class is going to initialize a `MockMVC` component and the test methods are calling the HTTP endpoint instead of direct method calls.

| Base class  | Test class |
| ------------- | ------------- |
| ![Controller class with endpoints](images/sample_controller_base.png)  | ![Test class for Controller with endpoints](images/sample_controller_test.png)  |

> Note, it's better keeping these tests in a separate test class considering the "integrated flavour" and that these a running slightly slower than the other tests.
> The checking of the results can be also trickier than with the mocked tests. Use the integrated tests only for checking the functionality that depends on the framework
> (i.e. is the endpoint available instead of resulting a HTTP 404 error, are the default values for the parameters correctly handled, the values correctly parsed from a JSon payload, etc).

# Release notes

## v1.2.9 bug fixes and configuration improvements
* fixed persistence of preferences
* preference for standard method bodies (before/after)
* TestUtils.assertTestFileEquals

## v1.2.8 bug fixes and configuration improvements
* fixed typo in easymock import
* preference for Assert.* vs Assertions.assertThat
* different hint for void methods for easymock vs mockito

## v1.2.7 improve assert generation
* assertion for Optional result
* assertion for ResponseEntity result
* preference page for default values

## v1.2.6 improve parameter value generation
* handling generic types
* assertion for Optional result
* assertion for Collection result

## v1.2.5 switch between code and test
* improved method name look-up

## v1.2.4 documentation and test generation improvements
* extended the readme for the use cases
* switch to test case works with the integration tests too
* option to generate new test method for same base method
* make "throws Exception" configurable
* link to issue tracker on GitHub
* fine-tune test method generation for
	* static methods
	* controller classes

## v1.2.3 sample test util
* sample test util implementation updates

## V1.2.2 rest controllers
* improved rest controller testing

## V1.2.1 enhanced test methods
* Boolean return value with assert isTrue
* Fixed static method test

## V1.2.0 basic functionality
* Generate test class  with Mockito (or EasyMock) Runner and Mock
* Generate test class with SpringRunner and MockBean
* Generate test methods for static methods
* Generate test method body with parameter initialization and isEqualTo assertion
