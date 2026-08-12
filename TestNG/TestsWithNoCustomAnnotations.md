## Test methods without Custom Annotations - to be processed per Property configuration

Create a custom annotation such that if @Test method is not annotated with "Custom Annotation" class add Custom annotation - TestInfo with values read from properties file and process test execution as if the test method was annotated with TestInfo annotation

**Java annotations** are immutable at runtime, you cannot physically add @TestInfo to a method after compilation. However, in an enterprise Selenium-TestNG framework you can implement a Virtual Annotation Injection Pattern where:

```text
                 TestNG Execution
                         │
                         ▼
              CustomAnnotationListener
                         │
                         ▼
             Is @TestInfo Available?
                    /         \
                  Yes          No
                   │            │
                   ▼            ▼
          Read Annotation    Read Defaults
              Values       From Properties File
                   │            │
                   └──────┬─────┘
                          ▼
                  TestMetadata Object
                          │
                          ▼
             Custom logic configuration for Retry / Reporting / Logging etc
                          │
                          ▼
                   Test Execution
```

