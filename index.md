# Ways to integrate Metrics Platform with PagerDuty

In the current use case, Metrics Platform and PagerDuty alert are mandatory. In this doc, three different ways of integrations will be discussed. The first two ways seems more viable. **These can and should be tested via a post request with raw body including JSON.** Testing through metrics platform is necessary but should be done after everything(message format) is set up. **PagerDuty abandons messages with undesired JSON style**.


## 1. Metrics Platform → Amazon SNS → PagerDuty ruleset → PagerDuty Alert (Current Setup)

### Pros:
- Set up already
- High availability by Amazon SNS
- No modification on Metrics Platform when creating a new ruleset

### Cons:
- Every alert is critical if not manually set it to info in the ruleset
- Summary cannot be set unless using regex in the ruleset. (Not difficult but have to perform the same action every time a ruleset is created)

---
### Example Output:
![](https://i.imgur.com/B3CkvIE.png)

### JSON Format:

```JSON
{
  "Description": "The nginx service is unresponsive.",
  
  "details": {
    "Trigger": {
      "Type": "Trigger",
      "Runbook": "https://wpengine.atlassian.net/wiki/spaces/SYS/pages/2193621272/Nginx+unresponsive",
      "Dashboard": "https://metrics-platform.wpesvc.net/d/vyMLFqPMz/corrections-service-dashboard?orgId=1&var-hostTotal=12&var-host=pod-140046",
      "Alert": "prodeng_templated_nginx_unresponsive"
    },
    "fdafsd": "<no value>",
    "StateChangeTime": "2021-06-25 00:56:00 +0000 UTC",
    "Region": "us-east1",
    "InstanceID": "",
    "Host": "pod-140046",
    "EventID": "net_response:host=pod-140046:prodeng_templated_nginx_unresponsive",
    "Description": "The nginx service is unresponsive.",
    "ClusterID": "140046"
  }
}
```

## 2. Metrics Platform → Amazon SNS → PagerDuty ruleset → PagerDuty Alert (Different setup)

### Pros:
- Semi-set up already
- High availability by Amazon SNS
- Fancy message format (Summary is shown and alert state level can be set)
- With a workaround, there is no need to create new Amazon SNS subscription everytime a new ruleset is created.
 1. Create a new ruleset in PagerDuty and copy its integration key.
 1. create a SNS https subscription to a ruleset → ```https://events.pagerduty.com/x-ere/[integration_key_here]``` and remember to enable raw message delivery.
 1. In the subscription tab **(Amazon SNS dashboard**), the subscribtion created should have a confirmed status.
 1. Create a new ruleset in PagerDuty which will be actually used with any rule needed, and copy its integration key.
 1. Use the above integreation key as the variable in the tickscript. 

### Cons:
- A routing key is mandatory to be included in the message sent by Metrics Platform. That means if any new message is desired to be sent to a new ruleset, update on the variable/template is necessary. 

---
### Example Output:
![](https://i.imgur.com/PmDqZqB.png)

### JSON Format:

```JSON
{
  "payload": {
    "summary": "Example alert on wpengine.com",
    "timestamp": "2015-07-17T08:42:58.315+0000",
    "source": "monitoringtool:cloudvendor:central-region-dc-01:852559987:cluster/api-stats-prod-003",
    "severity": "critical",
    "component": "postgres",
    "group": "prod-datapipe",
    "class": "deploy",
    "custom_details": {
      "ping time": "1500ms",
      "load avg": 0.75,
      "Trigger": {
      "Type": "Trigger",
      "Runbook": "https://wpengine.atlassian.net/wiki/spaces/SYS/pages/2193621272/Nginx+unresponsive",
      "Dashboard": "https://metrics-platform.wpesvc.net/d/vyMLFqPMz/corrections-service-dashboard?orgId=1&var-hostTotal=12&var-host=pod-140046",
      "Alert": "prodeng_templated_nginx_unresponsive"
    },
    "Anything": "<no value>",
    "StateChangeTime": "2021-06-25 00:56:00 +0000 UTC",
    "Region": "us-east1",
    "InstanceID": "",
    "Host": "pod-140046",
    "EventID": "net_response:host=pod-140046:prodeng_templated_nginx_unresponsive",
    "Description": "The nginx service is unresponsive.",
    "ClusterID": "140046"
    }
  },
  "routing_key": "R027TB4IDBUVC3N63VQ2HWHE90Q6TKUJ",
  "images": [
    {
      "src": "https://www.pagerduty.com/wp-content/uploads/2016/05/pagerduty-logo-green.png",
      "href": "https://example.com/",
      "alt": "Example text"
    }
  ],
  "links": [
    {
      "href": "https://example.com/",
      "text": "Link text"
    }, 
    {
      "href": "https://example.com/",
      "text": "Link text"
    }
  ],
  "event_action": "trigger",
  "client": "Sample Monitoring Service",
  "client_url": "https://monitoring.example.com"
} 

```


## 3. Metrics Platform → PagerDuty Alert

### Pros:
- Easy to set up
- May be more instanat since there is no mid point.
- Everything is well formatted including summary

### Cons:
- Availability may be an issue
- All modifications are done in metrics platform
- Impossible to filter by keyword through PagerDuty ruleset
