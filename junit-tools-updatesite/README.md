# JUnit Tools for Spring
[Update Site URL](https://csorbazoli.github.io/junit-tools-updatesite/)

## Aim of this project
Create a tool for helping the developers to write simple and maintainable unit tests with minimal effort on the tedious part and let them focus on the important part of the tests.

This plugin promotes the "Test First" approach. Create a well structured test method skeleton from the method signature with
* given-when-then blocks (Gherkin style)
** given: pre-initialized parameter values
** when: method is called
** then: assert for return value

Additional features:
* using mockito runner for fast test execution
* mock dependencies of the tested class
* using TestValueFactory for effortless but consistent input value creation
* using JSON comparison for checking complex result types instead of endless asserts on each single field value
* customization through preferences

Spring support:
* use Spring testing annotations if that is preferred or needed
* enhanced rest endpoint testing with MockMvc

## Samples
Base class

Test class

# Release notes

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
