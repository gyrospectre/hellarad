# Hella Rad!

![ooh so rad](https://media.giphy.com/media/l0MYylLtnC1ADCGys/giphy.gif)

Easy alert enrichment for overworked security teams! Designed to be modular and extensible, it will consume your alerts, enrich them with information that helps you triage quicker, and then feed the juicy results back into your alert pipeline (or ticketing system).

The only pre-requisite is that you must have an AWS account to host HellaRad. Currently, we support Splunk or OpsGenie as alert sources, and Jira or OpsGenie as output providers.

## How could I use this?

As an example, let's say that your security team uses Splunk for alerting and investigation, and Atlassian Jira for ticketing. By using the SNS alert action in the free Splunk Add-on for AWS, you can set your alerts to send to Hella Rad, which will take the results you define as interesting, extract any public IP addresses from them, and then run them through a bunch of services to get information about then. Helle Rad will then create a Jira ticket for your alert, and add this information as comments. Woot.

## Quick Start
1. Deploy HellaRad (see below)
2. Install the Splunk Add-on for AWS (https://splunkbase.splunk.com/app/1876/), to give you an SNS alert action. Configure the app with some AWS creds.
3. Update one of your saved searches, adding a `strcat` at the end to combine all the fields you think are of use to a new field called `interesting`.

`<awesome detection logic> | stats values(src_ip) as src_ip by dest_user | eval Detection="A test alert" | strcat src_ip "," dest_user interesting`

4. Add an 'AWS SNS Alert' action to your scheduled search, updating the 'Message' field of the action to `$result.interesting$`. Also fill out the Account and Region fields per the doco for the AWS Tech Add-on. The topic should be set to `hellarad-Alert`.

Next time this alert fires, the interesting fields will be sent to HellaRad, and the results sent to whatever you specify.

## Testing

```
sam build
sam local invoke IPAPIFunction --event event/alert.json
sam local invoke GreynoiseFunction --event event/alert.json
sam local invoke ConductorFunction --event event/sns.json 
sam local invoke OutputFunction --event event/output.json 
```