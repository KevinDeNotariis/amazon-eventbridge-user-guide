# Transforming Amazon EventBridge target input<a name="eb-transform-target-input"></a>

You can customize the text from an [event](eb-events.md) before EventBridge passes the event to the [target](eb-targets.md) of a [rule](eb-rules.md)\. Using the input transformer in the console or the API, you define variables that use JSON path to reference values in the original event source\. You can define up to 100 variables, assigning each a value from the input\. Then you can use those variables in the *Input Template* as <*variable\-name*>\. 

For a tutorial on using input transformer, see [Tutorial: Use input transformer to customize what EventBridge passes to the event target](eb-input-transformer-tutorial.md)\.

**Topics**
+ [Predefined variables](#eb-transform-input-predefined)
+ [Input Transform Examples](#eb-transform-input-examples)
+ [Transforming input by using the EventBridge API](#eb-transform-input-api)
+ [Transforming input by using AWS CloudFormation](#eb-transform-input-cfn)
+ [Common Issues with Transforming Input](#eb-transform-input-issues)

## Predefined variables<a name="eb-transform-input-predefined"></a>

There are pre\-defined variables you can use without defining a JSON path\. These variables are reserved, and you can't create variables with these names:
+ ****`aws.events.rule-arn` — The Amazon Resource Name \(ARN\) of the EventBridge rule\. 
+ ****`aws.events.rule-name` — The Name of the EventBridge rule\. 
+ ****`aws.events.event` — A copy of the original event\. 
+ ****`aws.events.event.ingestion-time` — The time at which the event was received by EventBridge\. This variable is generated by EventBridge and can't be overwritten\.
+ ****`aws.events.event.json` — The exact payload of an event as a string\.

## Input Transform Examples<a name="eb-transform-input-examples"></a>

The following is an example Amazon EC2 event\.

```
{
  "version": "0",
  "id": "7bf73129-1428-4cd3-a780-95db273d1602",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "123456789012",
  "time": "2015-11-11T21:29:54Z",
  "region": "us-east-1",
  "resources": [
    "arn:aws:ec2:us-east-1:123456789012:instance/i-abcd1111"
  ],
  "detail": {
    "instance-id": "i-0123456789",
    "state": "RUNNING"
  }
}
```

When defining a rule in the console, select the **Input Transformer** option under **Configure input**\. This option displays two text boxes: one for the *Input Path* and one for the *Input Template*\.

*Input Path* is used to define variables\. Use JSON path to reference items in your event and store those values in variables\. For instance, you could create an *Input Path* to reference values in the example event by entering the following in the first text box\.

```
{
  "timestamp" : "$.time",
  "instance" : "$.detail.instance-id", 
  "state" : "$.detail.state"
}
```

This defines two variables, `<instance>` and `<state>`\. You can reference these variables as you create your *Input Template*\.

The *Input Template* is a template for the information you want to pass to your target\. You can create a template that passes either a string or JSON to the target\. Using the previous event and *Input Path*, the following *Input Template* examples will transform the event to the example output before routing it to a target\.


| Description | Template | Output | 
| --- | --- | --- | 
| Simple string |  <pre>"instance <instance> is in <state>"</pre> |  <pre>"instance i-0123456789 is in RUNNING"</pre>  | 
|  **String with escaped quotes**  |  <pre>"instance \"<instance>\" is in <state>"</pre> |  <pre>"instance \"i-0123456789\" is in RUNNING"</pre> Note that this is the behavior in the EventBridge console\. The AWS CLI escapes the slash characters and the result is `"instance "i-0123456789" is in RUNNING"`\.  | 
|  **Simple JSON**  |  <pre>{<br />  "instance" : <instance>,<br />  "state": <state><br />}</pre> |  <pre>{<br />  "instance" : "i-0123456789",<br />  "state": "RUNNING"<br />}</pre>  | 
|  **JSON with strings and variables**  |  <pre>{<br /> "instance" : <instance>,<br /> "state": "<state>",<br /> "instanceStatus": "instance \"<instance>\" is in <state>"<br />}</pre>  |  <pre>{<br /> "instance" : "i-0123456789",<br /> "state": "RUNNING",<br /> "instanceStatus": "instance \"i-0123456789\" is in RUNNING"<br />}</pre>  | 
|  **JSON with a mix of variables and static information**  |  <pre>{<br />  "instance" : <instance>,<br />  "state": [ 9, <state>, true ],<br />  "Transformed" : "Yes"<br />}<br /></pre> |  <pre>{<br />  "instance" : "i-0123456789",<br />  "state": [<br />    9,<br />    "RUNNING",<br />    true<br />  ],<br />  "Transformed" : "Yes"<br />}</pre>  | 
|  **Including reserved variables in JSON**  |  <pre>{<br />  "instance" : <instance>,<br />  "state": <state>,<br />  "ruleArn" : <aws.events.rule-arn>,<br />  "ruleName" : <aws.events.rule-name>,<br />  "originalEvent" : <aws.events.event><br />}</pre> |  <pre>{<br />  "instance" : "i-0123456789",<br />  "state": "RUNNING",<br />  "ruleArn" : "arn:aws:events:us-east-2:123456789012:rule/example",<br />  "ruleName" : "example",<br />  "originalEvent" : {<br />    ... // commented for brevity<br />  }<br />}</pre>  | 
|  **Including reserved variables in a string**  | <pre>"<aws.events.rule-name> triggered"</pre> |  <pre>"example triggered"</pre>  | 
|  **Amazon CloudWatch log group**  | <pre>{<br />  "timestamp" : <timestamp>,<br />  "message": "instance \"<instance>\" is in <state>"<br />}</pre> |  <pre>{<br />  "timestamp" : 2015-11-11T21:29:54Z,<br />  "message": "instance "i-0123456789" is in RUNNING<br />}</pre>  | 

## Transforming input by using the EventBridge API<a name="eb-transform-input-api"></a>

For information about using the EventBridge API to transform input, see [Use Input Transformer to extract data from an event and input that data to the target](https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutTargets.html#API_PutTargets_Example_2)\.

## Transforming input by using AWS CloudFormation<a name="eb-transform-input-cfn"></a>

For information about using AWS CloudFormation to transform input, see [AWS::Events::Rule InputTransformer](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-events-rule-inputtransformer.html)\.

## Common Issues with Transforming Input<a name="eb-transform-input-issues"></a>

These are some common issues when transforming input in EventBridge:
+  For Strings, quotes are required\.
+  There is no validation when creating JSON path for your template\.
+  If you specify a variable to match a JSON path that doesn't exist in the event, that variable isn't created and won't appear in the output\.
+  The JSON that is passed to the target is minified and escaped\.
+  EventBridge doesn't escape values extracted by *Input Path*, when populating the *Input Template* for a target\.
