# csorbazoli.github.io
GitHub.io webpage for my repository

# JUnit Tools for Spring
[Update Site](https://csorbazoli.github.io/tree/main/junit-tools-updatesite)

## Aim of this project
Create a tool for helping the developers to write simple and maintainable unit tests with minimal effort on the tedious part and let them focus on the important part of the tests.

This plugin **promotes the "Test First" approach. Create a well structured test method skeleton from the method signature** with

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

**Find more details in the Release Notes...**

# JUnit Tools for Spring Release notes
Click link to display [Release Notes](https://github.com/csorbazoli/csorbazoli.github.io/junit-tools-updatesite/)
