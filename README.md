# ses-multitenant-reputation-management
Monitor and manage the SES sending reputation for a multi-tenant setup

**Background:**

Independent Software Vendors (ISVs) who have partners sending emails to their customers through their infrastructure, usually face the challenge of ensuring that all partners do not impact negatively their email sender reputation. Amazon Simple Email Service (SES) provides a reputation dashboard for bounces and complaints but this is on an account level rather than on a partner level. Furthermore SES might put your account under review if you exceed a certain threshold on bounce and complaint rates, which is again on an account level.

A desired state would allow ISVs to perform the actions below on a partner level:
1)  Monitor bounce and complaint rates
2)  Change the limits for bounce and complaint rates
3)  Pause the partner's ability to send emails and send them a notification if the bounce or complaint limit is exceeded

Having the above would provide ISVs visibility over their partners' email activity and prevent spillover of reputation damage from one partner to the wider SES account.

**Solution:**

Using other AWS services such as Kinesis Firehose, Lambda and DynamoDB a solution that will provide monitoring, threshold configuration, notifications and throttling per partner can be developed. Below you can find an architecture diagram of that solution:

![alt text](https://github.com/Pioank/ses-multitenant-reputation-management/blob/main/Images/SES-reputation-monitoring-management.JPG)

The process / data-flow of this solution is described below in steps:

1) In this solution the application that sends emails is hosted in a Lambda but it is not a requirement
2) Before the emails are sent on behalf of a partner, the application queries a DynamoDB database that hosts email KPIs per partner and assess if that partner's bounce and complaint rates are withing limits
3) If the partner's rates are within limits then the emails will be sent otherwise the application will not send the emails and inform the partner by sending an email to the email contact stored in DynamoDB
4) When sending the emails, each email will contain a tag where the partner ID will be stored. Read more about the send_email SES Boto3 SDK method - [here](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_email)
5) Email engagement data such as email open, clicked, bounces, complaints will be streamed via a Amazon Kinesis Firehose while a Lambda will perform further event processing
6) The Lambda will process the events: delivered, bounce and complaint which will be stored in a DynamoDB
7) The operators of this solution can amend the complaints_limit, bounces_limit and contact_email either from DynamoDB console or by developing a front end
