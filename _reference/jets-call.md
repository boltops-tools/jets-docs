---
title: jets call
reference: true
---

## Usage

    jets call [function] [event]

## Description

Call a lambda function on AWS or locally.

## Remote mode

Invoke the lambda function on AWS. The `jets call` command does a few extra things for your convenience:

It adds the function namespace to the function name.  So, you pass in `posts_controller-index` and the Lambda function is `demo-dev-posts_controller-index`.

CLI Name | Lambda Function
--- | ---
posts_controller-index | demo-dev-posts_controller-index

The `jets call` command can also print out the last 4KB of the lambda logs with the `--show-logs` option. The logging output is directed to stderr and the response output from the lambda function itself is directed to stdout so you can safely pipe the results of the call command to other tools like `jq`.

For [controllers](http://rubyonjets.com/docs/controllers/), the event you pass at the CLI is automatically transformed into [Lambda Proxy](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html) payload that contains the params as the `queryStringParameters`.  For example:

    {"test":1}

Gets changed to:

    {"queryStringParameters":{"test":1}}

This spares you from assembling the event payload manually to the payload that Jets controllers normally receive.  If you would like to disable this Lambda Proxy transformation then use the `--no-lambda-proxy` flag.

For [jobs](http://rubyonjets.com/docs/jobs/), the event is passed through untouched.

## Examples

    jets call hard_job-drive '{"test":1}'
    jets call hard_job-drive '{"test":1}' | jq .
    jets call hard_job-drive file://event.json | jq . # load event with a file
    jets call posts_controller-index # event payload is optional
    jets call posts_controller-index '{"path":"/posts"}' --show-log | jq .
    jets call posts_controller-index 'file://event.json' --show-log | jq .

The equivalent AWS Lambda CLI command:

    aws lambda invoke --function-name demo-dev-hard_job-dig --payload '{"path":"/posts","test":1}' outfile.txt
    cat outfile.txt | jq '.'

    aws lambda invoke --function-name demo-dev-posts_controller-index --payload '{"queryStringParameters":{"path":"/posts",test":1}}' outfile.txt
    cat outfile.txt | jq '.'

For convenience, you can also provide the function name with only dashes and `jets call` figures out what function you intend to call. Examples:

    jets call posts-controller-index
    jets call admin-related-pages-controller-index

Are the same as:

    aws lambda invoke --function-name demo-dev-posts_controller-index
    aws lambda invoke --function-name demo-dev-admin/related_pages_controller-index

Jets figures out what functions to call by evaluating the app code and finds if the controller and method exists.  If you want to turn guess mode off and want to always explicitly provide the method name use the `--no-guess` option.  The function name will then have to match the lambda function without the namespace. Example:

    jets call admin-related_pages_controller-index --no-guess

If you want to call a function which runs too long time, you can set `read_timeout`.

    jets call some_long_job-index --read_timeout 900
    
And you can set `retry_limit`. If you don't want to retry you can set 0.

    jets call some_long_job-index --retry_limit 0

## Local mode

Instead of calling AWS lambda remote, you can also have `jets call` use the code directly on your machine.  To enable this, use the `--local` flag. Example:

    jets call hard_job-drive --local

## Options

```
[--invocation-type=INVOCATION_TYPE]    # RequestResponse, Event, or DryRun
                                       # Default: RequestResponse
[--log-type=LOG_TYPE]                  # Works if invocation_type set to RequestResponse
                                       # Default: Tail
[--qualifier=QUALIFIER]                # Lambda function version or alias name
[--show-log], [--no-show-log]          # Shows last 4KB of log in the x-amz-log-result header
[--lambda-proxy], [--no-lambda-proxy]  # Enables automatic Lambda proxy transformation of the event payload
                                       # Default: true
[--guess], [--no-guess]                # Enables guess mode. Uses inference to allows use of all dashes to specify functions. Guess mode verifies that the function exists in the code base.
                                       # Default: true
[--local], [--no-local]                # Enables local mode. Instead of invoke the AWS Lambda function, the method gets called locally with current app code. With local mode guess mode is always used.
[--retry-limit=N]                      # Retry count of invoking function. It work with remote call
[--read-timeout=N]                     #  The number of seconds to wait for response data. It work with remote call
[--noop], [--no-noop]                  
```

