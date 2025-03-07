# Sending and receiving Amazon EventBridge events between event buses in the same account and Region<a name="eb-bus-to-bus"></a>

You can configure EventBridge to send and receive [events](eb-events.md) between [event buses](eb-event-bus.md) in the same AWS account and Region\. 

When you configure EventBridge to send or receive events between event buses, you use IAM roles on the *sender* event bus to give the *sender* event bus permission to send events to the *receiver* event bus\. You use [Resource\-based](eb-use-resource-based.md) policies on the *receiver* event bus to give the *receiver* event bus permission to receive events from the *sender* event bus\. You can also allow or deny events from certain event buses, specific [rules](eb-rules.md) associated with the event bus, or events from specific sources\. For more information about event bus permissions, including example policies, see [Permissions for Amazon EventBridge event buses](eb-event-bus-perms.md)\.

The steps to configure EventBridge to send events to or receive events between event buses in your account include the following:
+ To use an existing IAM role, you need to give either the sender event bus permissions to the receiver event bus or the receiver event bus permissions to the sender event bus\.
+ On the *sender* event bus, set up one or more rules that have the receiver event bus as the target and create an IAM role\.
+ On the *receiver* event bus, edit the permissions to allow events to be passed from the other event bus\.
+ On the *receiver* event, set up one or more rules that match events that come from the sender event bus\.
**Note**  
EventBridge can't route events received from a sender event bus to a third event bus\.

Events sent from one event bus to another are charged as custom events\. For more information, see [Amazon EventBridge Pricing](https://aws.amazon.com/eventbridge/pricing/)\.

To send events to another event bus, you create a rule with an event bus as a target\.

**To create a rule that sends events to a different event bus using the console**

1. Open the Amazon EventBridge console at [https://console\.aws\.amazon\.com/events/](https://console.aws.amazon.com/events/)\.

1. In the navigation pane, choose **Rules**\.

1. Choose **Create rule**\.

1. Enter a name and description for the rule\.

   A rule can't have the same name as another rule in the same Region and on the same event bus\.

1. For **Event bus**, choose the event bus that you want to associate with this rule\. If you want this rule to match events that come from your account, select ** AWS default event bus**\. When an AWS service in your account emits an event, it always goes to your account’s default event bus\.

1. For **Rule type**, choose **Rule with an event pattern**\.

1. Choose **Next**\.

1. For **Event source**, choose **AWS services**\.

1. \(Optional\) For **Sample events**, choose the type of event\.

1. For **Event pattern**, do one the following:
   + To use a template to create your event pattern, choose **Event pattern form** and choose the **Event source** and **Event type**\. If you choose **All Events** as the event type, all events emitted by this AWS service will match the rule\.

     To customize the template, choose **Custom pattern \(JSON editor\)** and make your changes\.
   + To use a custom event pattern, choose **Custom pattern \(JSON editor\)** and create your event pattern\.

1. Choose **Next**\.

1. For **Target types**, choose **EventBridge event bus** and choose **Event bus in the same AWS account and Region**\.

1. For **Event bus as a target**, select an event bus from the drop\-down list\.

1. For **Execution role**, to have EventBridge create a new IAM role that has permissions to send events to the specified event bus, choose **Create a new role for this specific resource**\.

1. \(Optional\) For **Additional settings**, do the following:

   1. For **Dead\-letter queue**, choose whether to use a standard Amazon SQS queue as a dead\-letter queue\. EventBridge sends events that match this rule to the dead\-letter queue if they are not successfully delivered to the target\. Do one of the following:
     + Choose **None** to not use a dead\-letter queue\.
     + Choose **Select an Amazon SQS queue in the current AWS account to use as the dead\-letter queue** and then select the queue to use from the drop\-down list\.
     + Choose **Select an Amazon SQS queue in an other AWS account as a dead\-letter queue** and then enter the ARN of the queue to use\. You must attach a resource\-based policy to the queue that grants EventBridge permission to send messages to it\. For more information, see [Granting permissions to the dead\-letter queue](eb-rule-dlq.md#eb-dlq-perms)\.

1. \(Optional\) Choose **Add another target** to add another target for this rule\.

1. Choose **Next**\.

1. \(Optional\) Enter one or more tags for the rule\. For more information, see [Amazon EventBridge tags](eb-tagging.md)\.

1. Choose **Next**\.

1. Review the details of the rule and choose **Create rule**\.