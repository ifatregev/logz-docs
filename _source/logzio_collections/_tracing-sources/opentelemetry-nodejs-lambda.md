---
title: Sending traces from your Node.js application on AWS Lambda using OpenTelemetry
logo:
  logofile: lambda-nodejs2.png
  orientation: vertical
data-source: Automatic instrumentation for Node.js & Lambda
data-for-product-source: Tracing
templates: ["network-device-filebeat"]
contributors:
  - nshishkin
  - yotamloe
shipping-tags:
  - new-instrumentation
order: 1380
---

Deploy this integration to auto-instrument your Node.js application running on AWS Lambda and send the traces to your Logz.io account. This is done by adding a dedicated layer for OpenTelemetry collector, a dedicated layer for Node.js auto-instrumentation and configuring environment variables of these layers. This integration will require no change to your application code.

<!-- info-box-start:info -->
This integration only works for the following AWS regions: us-east-1, us-east-2, ca-central-1, ap-northeast-2, ap-northeast-1, eu-central-1, eu-west-2.
{:.info-box.note}
<!-- info-box-end -->

#### Setup instructions

**Before you begin, you'll need**:
  
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
* Configured [AWS credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
* A Lambda function with a Node.js application that is not yet instrumented.

<!-- info-box-start:info -->
Adding environmental variables using the AWS CLI commands below, will overwrite all existing variables for your Lambda function.
{:.info-box.note}
<!-- info-box-end -->

<!-- info-box-start:info -->
This integration uses OpenTelemetry Collector Contrib, not the OpenTelemetry Collector Core.
{:.info-box.important}
<!-- info-box-end -->

<div class="tasklist">

##### Add the OpenTelemetry collector layer to your Lambda function 

This layer contains the OpenTelemetry collector that will capture data from your application.

```shell
aws lambda update-function-configuration --function-name <<YOUR-LAMBDA_FUNCTION_NAME>> --layers arn:aws:lambda:<<YOUR-AWS-REGION>>:486140753397:layer:logzio-opentelemetry-collector-layer:1
```

Replace `<<YOUR-LAMBDA_FUNCTION_NAME>>` with the name of your Lambda function running the Node.js application.

Replace `<<YOUR-AWS-REGION>>` with the code of your AWS regions, e.g. `us-east-1`.

##### Create a configuration file for the OpenTelemetry collector
  
By default, the OpenTelemetry collector layer exports data to the Lambda console. To customize the collector configuration, you need to add a `collector.yaml` to your function and specifiy its location via the `OPENTELEMETRY_COLLECTOR_CONFIG_FILE` environment variable.


The `collector.yaml` file will have the following configuration:
  
```yaml
receivers:
  jaeger:
    protocols:
      thrift_compact:
        endpoint: "0.0.0.0:6831"
      thrift_binary:
        endpoint: "0.0.0.0:6832"
      grpc:
        endpoint: "0.0.0.0:14250"
      thrift_http:
        endpoint: "0.0.0.0:14268"
  opencensus:
    endpoint: "0.0.0.0:55678"
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
      http:
        endpoint: "0.0.0.0:4318"
  zipkin:
    endpoint: "0.0.0.0:9411"


exporters:
  logzio/traces:
    account_token: "<<TRACING-SHIPPING-TOKEN>>"
    region: "<<LOGZIO_ACCOUNT_REGION_CODE>>"

  logging:

processors:
  batch:
  tail_sampling:
    policies:
      [
        {
          name: policy-errors,
          type: status_code,
          status_code: {status_codes: [ERROR]}
        },
        {
          name: policy-slow,
          type: latency,
          latency: {threshold_ms: 1000}
        }, 
        {
          name: policy-random-ok,
          type: probabilistic,
          probabilistic: {sampling_percentage: 10}
        }        
      ]

extensions:
  pprof:
    endpoint: :1777
  zpages:
    endpoint: :55679
  health_check:

service:
  extensions: [health_check, pprof, zpages]
  pipelines:
    traces:
      receivers: [opencensus, jaeger, zipkin, otlp]
      processors: [tail_sampling, batch]
      exporters: [logging, logzio/traces]
```

  
{% include /tracing-shipping/replace-tracing-token.html %}
{% include /tracing-shipping/tail-sampling.md %}


##### Direct the OpenTelemetry collector to the configuration file


Add `OPENTELEMETRY_COLLECTOR_CONFIG_FILE` variable to direct the OpenTelemetry collector to the configuration file:

```
aws lambda update-function-configuration --function-name <<YOUR-LAMBDA_FUNCTION_NAME>> --environment Variables={OPENTELEMETRY_COLLECTOR_CONFIG_FILE=/var/task/collector.yaml}
```

Replace `<<YOUR-LAMBDA_FUNCTION_NAME>>` with the name of your Lambda function running the Node.js application.


##### Activate tracing for your Lambda function

```shell
aws lambda update-function-configuration --function-name <<YOUR-LAMBDA_FUNCTION_NAME>> --tracing-config Mode=Active
```

Replace `<<YOUR-LAMBDA_FUNCTION_NAME>>` with the name of your Lambda function running the Node.js application.

##### Add the OpenTelemetry Node.js wrapper layer to your Lambda function

The OpenTelemetry Node.js wrapper layer automatically instruments the Node.js application in your Lambda function.

```shell
aws lambda update-function-configuration --function-name <<YOUR-LAMBDA_FUNCTION_NAME>> --layers arn:aws:lambda:<<YOUR-AWS-REGION>>:486140753397:layer:logzio-opentelemetry-nodejs-wrapper:1
```

Replace `<<YOUR-LAMBDA_FUNCTION_NAME>>` with the name of your Lambda function running the Node.js application.

Replace `<<YOUR-AWS-REGION>>` with the code of your AWS regions, e.g. `us-east-1`.
  
##### Add environment variable for the wrapper
  
Add the `AWS_LAMBDA_EXEC_WRAPPER` environment variable to point to the `otel-handler` executable:

```shell
aws lambda update-function-configuration --function-name <<YOUR-LAMBDA_FUNCTION_NAME>> --environment Variables={AWS_LAMBDA_EXEC_WRAPPER=/opt/otel-handler}
```

Replace `<<YOUR-LAMBDA_FUNCTION_NAME>>` with the name of your Lambda function running the Node.js application.

  
</div>
