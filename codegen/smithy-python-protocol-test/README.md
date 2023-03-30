## Smithy Python Protocol Test

This package generates and runs [protocol tests
](https://smithy.io/2.0/additional-specs/http-protocol-compliance-tests.html)
for all supported protocols. Protocol tests are defined in the Smithy model itself
using traits, and are generated along with the client. These test the actual
functionality of the generated client and protocol implementation using expected
inputs and outputs. The tests are generated by [`HttpProtocolTestGenerator`
](https://github.com/awslabs/smithy-python/blob/develop/codegen/smithy-python-codegen/src/main/java/software/amazon/smithy/python/codegen/HttpProtocolTestGenerator.java).

For AWS protocols, these tests are defined [here
](https://github.com/awslabs/smithy/tree/main/smithy-aws-protocol-tests).

To run these tests, run `make test-protocols` from the repository root, ensuring that
you have a [Python virtual environment](https://docs.python.org/3/library/venv.html)
active.

### When should I change this package?

This package should only be changed when support for a new protocol is initially
introduced.

### How can I add new protocol tests to an exiting protocol?

To add new protocol tests, the source protocol test package must be updated. For
example, AWS protocol tests must be defined in the [smithy-aws-protocol-tests
](https://github.com/awslabs/smithy/tree/main/smithy-aws-protocol-tests) package.
After a new test case is added to the Smithy model, publish the change locally by
running `./gradlew build publishToMavenLocal`. Make sure that the version range of the
dependency here contains the version of the protocol test package you are building.

### How can I add protocol tests for a new protocol?

First, add a dependency on the package that contains the tests for the protocol. For
AWS protocols, that already exists. Next, add a new [projection
](https://smithy.io/2.0/guides/building-models/build-config.html#projections)
in `smithy-build.json` for each service that has tests for the protocol. For example:

```json
{
    "version": "1.0",
    "projections": {
        "rest-json-1": {
            "transforms": [{
                "name": "includeServices",
                "args": {
                    "services": ["aws.protocoltests.restjson#RestJson"]
                }
            }],
            "plugins": {
                "python-codegen": {
                    "service": "aws.protocoltests.restjson#RestJson",
                    "module": "restjson",
                    "moduleVersion": "0.0.1"
                }
            }
        },
        "my-new-protocol": {
            "transforms": [{
                "name": "includeServices",
                "args": {
                    "services": ["com.example#MyNewProtocolTestService"]
                }
            }],
            "plugins": {
                "python-codegen": {
                    "service": "com.example#MyNewProtocolTestService",
                    "module": "mynewprotocol",
                    "moduleVersion": "0.0.1"
                }
            }
        }
    }
}
```