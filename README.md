## Issue
When using DynamoDB with the url-connection-client and explicitly excluding netty and apache async clients, attempting to inject `DynamoDBClient` results in an error which implies SdkAsyncHttpClient is missing. This is incorrect as we're attempting to instantiate the synchronous version of the DynamoDB client instead of the async (netty/apache based) one.

## Fix
It appears that the `@Requires(beans = SdkAsyncHttpClient.class)` is missing the async factory methods in io.micronaut.aws.sdk.v2.service.dynamodb.DynamoDbClientFactory in micronaut-aws. This annotation is present on the relevant async methods in S3, SNS, SQS factories but missing from the DynamoDB one. Adding the change locally and retesting resulted in the attached unit test passing.

## How to Run & Reproduce Error
Run `./gradlew clean build` - the test case should fail with the following stack trace:
```
o.micronaut.context.exceptions.BeanInstantiationException: Bean definition [software.amazon.awssdk.services.dynamodb.DynamoDbAsyncClientBuilder] could not be loaded: Failed to inject value for parameter [httpClient] of class: software.amazon.awssdk.services.dynamodb.DynamoDbAsyncClientBuilder

Message: No bean of type [software.amazon.awssdk.http.async.SdkAsyncHttpClient] exists. Make sure the bean is not disabled by bean requirements (enable trace logging for 'io.micronaut.context.condition' to check) and if the bean is enabled then ensure the class is declared a bean and annotation processing is enabled (for Java and Kotlin the 'micronaut-inject-java' dependency should be configured as an annotation processor).
```
