---

copyright:
  years: 2020
lastupdated: "2020-08-31"

keywords: resource configuration, resource governance, governance, rule, config rule, properties, conditions, enforcement actions, evaluation results

subcollection: security-compliance

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:download: .download}
{:external: target="_blank" .external}
{:new_window: target="_blank"}
{:faq: data-hd-content-type='faq'}
{:gif: data-image-type='gif'}
{:term: .term}
{:important: .important}
{:note: .note}
{:pre: .pre}
{:tip: .tip}
{:preview: .preview}
{:deprecated: .deprecated}
{:shortdesc: .shortdesc}
{:support: data-reuse='support'}
{:script: data-hd-video='script'}
{:table: .aria-labeledby="caption"}
{:beta: .beta}


# Working with config rules
{: #rules}

A config rule is a JSON document that defines the configuration of resources. With the {{site.data.keyword.compliance_full}}, you can create rules for specific {{site.data.keyword.cloud_notm}} resource types to govern the way that resources in your accounts can be provisioned or configured.
{: shortdesc}

After you create a rule, you can monitor for configuration changes by attaching the rule to a scope, such as an account group, specific accounts, or an entire enterprise. You can further investigate noncompliant resources by reviewing your evaluation results in the {{site.data.keyword.compliance_short}} UI.

To work with config rules, you must have an {{site.data.keyword.cloud_notm}} enterprise account. Currently, you can create rules for managing [public access to account resources](/docs/account?topic=account-public).
{: note}

To learn more about the different components of a rule, check out the following example.

![The image shows the components of the configuration rule, including its target, required configuration, and enforcement actions. The target represents the IBM Cloud service and resource type that you want to target. The required configuration section details the property checks and conditions that you want to apply. The enforcement actions are the actions that IBM takes on your behalf in cases of noncompliance.](images/config-rules.svg){: caption="Figure 1. Example configuration rule" caption-side="bottom"}

## Before you begin
{: #before-rules}

Before you get started, be sure that you have the following prerequisites.

- An {{site.data.keyword.cloud_notm}} enterprise account.
- The required level of access to view and manage rules. To create a rule, you need the editor platform role or higher. For more information, see [Assigning access](/docs/security-compliance?topic=security-compliance-access-management).

## Creating rules
{: #create-rules}

You can use the {{site.data.keyword.compliance_short}} UI or API to define the rules that you want to enforce or monitor for your {{site.data.keyword.cloud_notm}} resources.

### Creating rules in the console
{: #create-rules-ui}

To create rules by using the {{site.data.keyword.cloud_notm}} console:

1. In the {{site.data.keyword.cloud_notm}} console, click the **Menu** icon ![Menu icon](../icons/icon_hamburger.svg) > **Security and Compliance**.
2. In the navigation, click **Configure rules**.
3. Click **Create**.
4. Give your rule a meaningful name and description.
5. Optional: Add one or more labels that you can use to organize and search for similar rules.
6. Click **Next**.
7. Select the service and resource kind that you want to target.

  Currently, you can create rules that target the IAM Access Groups Service.
  {: note} 
6. Use the JSON editor to set configuration properties for the rule.

  The following JSON snippet shows an example rule definition that checks to ensure that [public access to account resources](/docs/account?topic=account-public) is disabled.

  ```json
  {
    "target": {
      "service_name": "iam-groups",
      "resource_kind": "service"
    },
    "required_config": {
      "description": "Public access check",
      "and": [
        {
          "property": "public_access_enabled",
          "operator": "is_false"
        }
      ]
    }
  }
  ```
  {: codeblock}
6. Click **Next**.
7. Select the enforcement actions that you want to apply.

  Enforcement actions define how the {{site.data.keyword.compliance_short}} reacts when a resource is created or modified and the requested configuration is not compliant with your rule. The following table describes the supported actions.

  <table>
    <caption>Table 1. Enforcement actions and descriptions</caption>
    <tr>
      <th>Action</th>
      <th>Description</th>
    </tr>
    <tr>
      <td>Disallow</td>
      <td>Blocks the requested action from completing.</td>
    </tr>
    <tr>
      <td>Audit log</td>
      <td>Allows the requested action to complete, but logs an event that indicates noncompliance in {{site.data.keyword.at_short}}.</td>
    </tr>
  </table>

  Enforcement actions are triggered only when an account user creates or modifies the resource that you are targeting with a rule.
  {: note}
11. Review your selections. To make an update, click **Back** to return to the section that you want to edit.
12. Click **Finish and attach** to create your rule and [attach it to a scope](#evaluate-rules). 

    If you're not ready to attach your rule, you can always view your rules and create an attachment later.

### Creating a rule by using the API
{: #create-rule-api}

You can create rules programmatically by using the {{site.data.keyword.compliance_short}} API.

The following example request creates a rule to ensure that [public access to account resources](/docs/account?topic=account-public) is disabled. 

```bash
curl -X POST \
"https://compliance.{DomainName}/config/v1/rules" \
  -H 'Authorization: Bearer <access_token>' \
  -H 'Content-type: application/json' \
  -H 'Transaction-Id': 'a7f48341-a2b0-4649-a95d-d416d5fb4170' \
  -d '{
  "rules": [
    {
      "request_id": "80e8c8ce-06ea-44dd-8a45-3e33293ddd78",
      "rule": {
        "account_id": "<account_ID>",
        "name": "Disable public access",
        "description": "Ensure that public access to account resources is disabled.",
      "target": {
        "service_name": "iam-groups",
        "resource_kind": "service",
      "required_config": {
        "description": "Public access property check",
        "and": [
          {
            "property": "public_access_enabled",
            "operator": "is_false"          
          {
        ],
      "enforcement_actions": [
        {
          "action": "disallow"
        },
        {
          "action": "audit_log"
        }
      ],
      "labels": [
        "SOC2",
        "ITCS300"
      ]
    }
```
{: codeblock}

A successful `POST config/v1/rules` response returns the ID value for your rule, along with other metadata. For more information about the required and optional request parameters, see [Create rules](/apidocs/security-compliance/config#create-rules).

## Viewing rules
{: #view-rules}

After you create a rule, you can view it by navigating to the {{site.data.keyword.compliance_short}} UI or by using the API. 

### Viewing rules in the console
{: #view-rules-ui}

To view rules by using the {{site.data.keyword.compliance_short}} UI:

1. In the {{site.data.keyword.cloud_notm}} console, click the **Menu** icon ![Menu icon](../icons/icon_hamburger.svg) **> Security and Compliance**.
2. In the navigation, click **Configure rules**.

   From the **Configuration rules** table, you can see a list of your available rules and the {{site.data.keyword.cloud_notm}} service that is associated with each rule. You can also view the enforcement actions that are associated with each rule in cases of noncompliance.
3. From the list of rules, click the name of the rule that you want to view.

   You can view the parameters that are associated with the rule. To view or edit the [scope attachments](#evaluate-rules) for the rule, click **Attachments** in the navigation.

### Viewing rules by using the API
{: #view-rules-api}

The following request example lists the rules that are available in your account. 

```bash
curl -X POST \
"https://compliance.{DomainName}/config/v1/rules?account_id=<account_ID>" \
  -H 'Authorization: Bearer <access_token>' \
  -H 'Content-type: application/json'
```
{: codeblock}

A successful `GET config/v1/rules` response returns the IDs values for your available rules, along with other summary details. For more information about the required and optional request parameters, see [List rules](/apidocs/security-compliance/config#list-rules).

## Attaching rules to scopes
{: #evaluate-rules}

After you create a rule, you can enforce and evaluate it by using the {{site.data.keyword.compliance_short}}.

By creating an attachment between a rule and a scope, you can monitor the resources that exist in that scope against the rule that you defined. If you have a specific account that you don't want the rule to apply to, you can choose to exclude it. For example, if you apply a rule to an entire enterprise you might want to exclude a scope that is used primarily for testing.

### Attaching a rule to a scope in the console
{: #evaluate-rules-ui}

To create an attachment for an existing rule by using the {{site.data.keyword.compliance_short}} UI:

1. In the {{site.data.keyword.cloud_notm}} console, click the **Menu** icon ![Menu icon](../icons/icon_hamburger.svg) **> Security and Compliance**.
2. In the navigation, click **Configure rules**.
3. From the list of rules, click the name of the rule that you want to attach.
4. Click **Attachments**. If the rule that you selected is attached to any scopes, you see them in a table.
5. Click **Attach** to review your attachment options.
     1. Decide whether to apply the rule across the entire enterprise or narrow it to a specific account.
     2. Review the hierarchy of the account or enterprise that you selected.
     3. If you want to exclude a scope, switch the toggle to **Yes**. Select the scopes that you want to exclude and click **Add**.
6. Click **Attach**.

    Now that your rule is attached to a scope, the scope is scanned for possible noncompliance the next time that a scan is scheduled to run. Updated reports are generated automatically for your resources once every 24 hours.

### Attaching a rule to a scope by using the API
{: #evaluate-rules-api}

The following example request creates an attachment between an existing rule and a scope. 

```bash
curl -X POST \
"https://compliance.{DomainName}/config/v1/rules/<rule_ID>/attachments" \
  -H 'Authorization: Bearer <access_token>' \
  -H 'Content-type: application/json' \
  -d '{
  "attachments": [
    {
      "included_scope": {
        "scope_id": "<included_scope_ID>",
        "scope_type": "<scope_type>",
        "note": "<description>"
      },
      "excluded_scopes": [
        {
          "scope_id": "<excluded_scope_ID",
          "scope_type": "<scope_type>",
          "note": "<description>"
        }
      ]
    }
  ]
}'
```
{: codeblock}

A successful `POST config/v1/rules/{rule_ID}/attachments` response returns the ID value for the attachment, along with other metadata. For more information about the required and optional request parameters, see [Create attachments](/apidocs/security-compliance/config#create-attachments). 

## Deleting rules
{: #delete-rules}

If you no longer need to use a rule, you can choose to delete it by using the {{site.data.keyword.compliance_short}} UI or API.

### Deleting rules in the console
{: #delete-rules-ui}

To delete an existing rule by using the {{site.data.keyword.compliance_short}} UI:

1. In the {{site.data.keyword.cloud_notm}} console, click the **Menu** icon ![Menu icon](../icons/icon_hamburger.svg) **> Security and Compliance**.
2. In the navigation, click **Configure rules**.
3. Use the **Configuration rules** table to browse the rules in your account.
4. Click the **Actions** icon ![Actions icon](../icons/actions-icon-vertical.svg) to open a list of options for the rule that you want to delete.
5. From the options menu, click **Delete** and confirm the deletion in the next screen.

### Deleting rules by using the API
{: #delete-rules-api}

The following example request deletes an existing rule. 

```bash
curl -X DELETE \
"https://compliance.{DomainName}/config/v1/rules/<rule_ID> \
  -H 'Authorization: Bearer <access_token>' 
  -H 'Content-type: application/json'
```
{: codeblock}

A successful `DELETE config/v1/rules/{rule_ID}` response returns a `204 No Content` status code to indicate that your rule was successfully deleted. For more information about the required and optional request parameters, see [Delete a rule](/apidocs/security-compliance/config#delete-a-rule). 
