---
title: Automate incident handling in Microsoft Sentinel | Microsoft Docs
description: This article explains how to use automation rules to automate incident handling, in order to maximize your SOC's efficiency and effectiveness in response to security threats.
author: yelevin
ms.topic: conceptual
ms.date: 11/09/2021
ms.author: yelevin
ms.custom: ignite-fall-2021
---

# Automate incident handling in Microsoft Sentinel with automation rules

[!INCLUDE [Banner for top of topics](./includes/banner.md)]

This article explains what Microsoft Sentinel automation rules are, and how to use them to implement your Security Orchestration, Automation and Response (SOAR) operations, increasing your SOC's effectiveness and saving you time and resources.

## What are automation rules?

Automation rules are a way to centrally manage the automation of incident handling, allowing you to perform simple automation tasks without using playbooks.

For example, automation rules allow you to automatically:
- Suppress noisy incidents.
- Triage new incidents by changing their status from New to Active and assigning an owner.
- Tag incidents to classify them.
- Escalate an incident by assigning a new owner.
- Close resolved incidents, specifying a reason and adding comments.

Automation rules can also:
- Automate responses for multiple analytics rules at once.
- Control the order of actions that are executed.
- Inspect the incident's contents (alerts, entities, and other properties) and take further action by calling a playbook.

In short, automation rules streamline the use of automation in Microsoft Sentinel, enabling you to simplify complex workflows for your incident orchestration processes.

## Components

Automation rules are made up of several components:

- **[Triggers](#triggers)** that define what kind of incident event will cause the rule to run, subject to...
- **[Conditions](#conditions)** that will determine the exact circumstances under which the rule will run and perform...
- **[Actions](#actions)** to change the incident in some way or call a [playbook](automate-responses-with-playbooks.md).

### Triggers

Automation rules are triggered **when an incident is created or updated** (the update trigger is now in **Preview**). Recall that incidents are created from alerts by analytics rules, of which there are several types, as explained in [Detect threats with built-in analytics rules in Microsoft Sentinel](detect-threats-built-in.md).

The following table shows the different possible ways that incidents can be created or updated that will cause an automation rule to run.

| Trigger type | Events that cause the rule to run |
| --------- | ------------ |
| **When incident is created** | - A new incident is created by an analytics rule.<br>- An incident is ingested from Microsoft 365 Defender.<br>- A new incident is created manually. |
| **When incident is updated**<br>(Preview) | - An incident's status is changed (closed/reopened/triaged).<br>- An incident's owner is assigned or changed.<br>- An incident's severity is raised or lowered.<br>- Alerts are added to an incident.<br>- Comments, tags, or tactics are added to an incident. |

### Conditions

Complex sets of conditions can be defined to govern when actions (see below) should run. These conditions include the event that triggers the rule (incident created or updated), the states or values of the incident's properties and [entity properties](entities-reference.md), and also the analytics rule or rules that generated the incident.

When an automation rule is triggered, it checks the triggering incident against the conditions defined in the rule. The property-based conditions are evaluated according to **the current state** of the property at the moment the evaluation occurs, or according to **changes in the state** of the property (see below for details). Since a single incident creation or update event could trigger several automation rules, the **order** in which they run (see below) makes a difference in determining the outcome of the conditions' evaluation. The **actions** defined in the rule will run only if all the conditions are satisfied.

#### Incident create trigger

For rules defined using the trigger **When an incident is created**, you can define conditions that check the **current state** of the values of a given list of incident properties, using one or more of the following operators:

An incident property's value 
- **equals** or **does not equal** the value defined in the condition.
- **contains** or **does not contain** the value defined in the condition.
- **starts with** or **does not start with** the value defined in the condition.
- **ends with** or **does not end with** the value defined in the condition.



The **current state** in this context refers to the moment the condition is evaluated - that is, the moment the automation rule runs. If more than one automation rule is defined to run in response to the creation of this incident, then changes made to the incident by an earlier-run automation rule are considered the current state for later-run rules.

#### Incident update trigger

The conditions evaluated in rules defined using the trigger **When an incident is updated** include all of those listed for the incident creation trigger. But the update trigger includes more properties that can be evaluated.

One of these properties is **Updated by**. This property lets you track the type of source that made the change in the incident. You can create a condition evaluating whether the incident was updated by one of the following:

- an application
- a user
- an alert grouping (that added alerts to the incident)
- a playbook
- an automation rule
- Microsoft 365 Defender

Using this condition, for example, you can instruct this automation rule to run on any change made to an incident, except if it was made by another automation rule.

More to the point, the update trigger also uses other operators that check **state changes** in the values of incident properties as well as their current state. A **state change** condition would be satisfied if:

An incident property's value was
- **changed** (regardless of the actual value before or after).
- **changed from** the value defined in the condition.
- **changed to** the value defined in the condition.
- **added** to (this applies to properties with a list of values).

> [!NOTE]
> - An automation rule, based on the update trigger, can run on an incident that was updated by another automation rule, based on the incident creation trigger, that ran on the incident.
>
> - Also, if an incident is updated by an automation rule that ran on the incident's creation, the incident can be evaluated by *both* a subsequent *incident-creation* automation rule *and* an *incident-update* automation rule, both of which will run if the incident satisfies the rules' conditions.
>
> - If an incident triggers both create-trigger and update-trigger automation rules, the create-trigger rules will run first, according to their **[Order](#order)** numbers, and then the update-trigger rules will run, according to *their* **Order** numbers.


### Actions

Actions can be defined to run when the conditions (see above) are met. You can define many actions in a rule, and you can choose the order in which they’ll run (see below). The following actions can be defined using automation rules, without the need for the [advanced functionality of a playbook](automate-responses-with-playbooks.md):

- Changing the status of an incident, keeping your workflow up to date.

  - When changing to “closed,” specifying the [closing reason](investigate-cases.md#closing-an-incident) and adding a comment. This helps you keep track of your performance and effectiveness, and fine-tune to reduce [false positives](false-positives.md).

- Changing the severity of an incident – you can reevaluate and reprioritize based on the presence, absence, values, or attributes of entities involved in the incident.

- Assigning an incident to an owner – this helps you direct types of incidents to the personnel best suited to deal with them, or to the most available personnel.

- Adding a tag to an incident – this is useful for classifying incidents by subject, by attacker, or by any other common denominator.

Also, you can define an action to [**run a playbook**](tutorial-respond-threats-playbook.md), in order to take more complex response actions, including any that involve external systems. **Only** playbooks activated by the [**incident trigger**](automate-responses-with-playbooks.md#azure-logic-apps-basic-concepts) are available to be used in automation rules. You can define an action to include multiple playbooks, or combinations of playbooks and other actions, and the order in which they will run.

Playbooks using [either version of Logic Apps (Standard or Consumption)](automate-responses-with-playbooks.md#two-types-of-logic-apps) will be available to run from automation rules.

### Expiration date

You can define an expiration date on an automation rule. The rule will be disabled after that date. This is useful for handling (that is, closing) "noise" incidents caused by planned, time-limited activities such as penetration testing.

### Order

You can define the order in which automation rules will run. Later automation rules will evaluate the conditions of the incident according to its state after being acted on by previous automation rules.

For example, if "First Automation Rule" changed an incident's severity from Medium to Low, and "Second Automation Rule" is defined to run only on incidents with Medium or higher severity, it won't run on that incident.

## Common use cases and scenarios

### Incident-triggered automation

Before automation rules existed, only alerts could trigger an automated response, through the use of playbooks. With automation rules, incidents can now trigger automated response chains, which can include new incident-triggered playbooks ([special permissions are required](#permissions-for-automation-rules-to-run-playbooks)), when an incident is created. 

### Trigger playbooks for Microsoft providers

Automation rules provide a way to automate the handling of Microsoft security alerts by applying these rules to incidents created from the alerts. The automation rules can call playbooks ([special permissions are required](#permissions-for-automation-rules-to-run-playbooks)) and pass the incidents to them with all their details, including alerts and entities. In general, Microsoft Sentinel best practices dictate using the incidents queue as the focal point of security operations.

Microsoft security alerts include the following:

- Microsoft Defender for Cloud Apps (formerly Microsoft Cloud App Security)
- Azure AD Identity Protection
- Microsoft Defender for Cloud (formerly Azure Defender or Azure Security Center)
- Defender for IoT (formerly Azure Security Center for IoT)
- Microsoft Defender for Office 365
- Microsoft Defender for Endpoint
- Microsoft Defender for Identity

### Multiple sequenced playbooks/actions in a single rule

You can now have near-complete control over the order of execution of actions and playbooks in a single automation rule. You also control the order of execution of the automation rules themselves. This allows you to greatly simplify your playbooks, reducing them to a single task or a small, straightforward sequence of tasks, and combine these small playbooks in different combinations in different automation rules.

### Assign one playbook to multiple analytics rules at once

If you have a task you want to automate on all your analytics rules – say, the creation of a support ticket in an external ticketing system – you can apply a single playbook to any or all of your analytics rules – including any future rules – in one shot. This makes simple but repetitive maintenance and housekeeping tasks a lot less of a chore.

### Automatic assignment of incidents

You can assign incidents to the right owner automatically. If your SOC has an analyst who specializes in a particular platform, any incidents relating to that platform can be automatically assigned to that analyst.

### Incident suppression

You can use rules to automatically resolve incidents that are known false/benign positives without the use of playbooks. For example, when running penetration tests, doing scheduled maintenance or upgrades, or testing automation procedures, many false-positive incidents may be created that the SOC wants to ignore. A time-limited automation rule can automatically close these incidents as they are created, while tagging them with a descriptor of the cause of their generation.

### Time-limited automation

You can add expiration dates for your automation rules. There may be cases other than incident suppression that warrant time-limited automation. You may want to assign a particular type of incident to a particular user (say, an intern or a consultant) for a specific time frame. If the time frame is known in advance, you can effectively cause the rule to be disabled at the end of its relevancy, without having to remember to do so.

### Automatically tag incidents

You can automatically add free-text tags to incidents to group or classify them according to any criteria of your choosing.

## Use cases added by update trigger

Now that changes made to incidents can trigger automation rules, more scenarios are open to automation. 

### Extend automation when incident evolves

You can use the update trigger to apply many of the above use cases to incidents as their investigation progresses and analysts add alerts, comments, and tags. Control alert grouping in incidents.

### Update orchestration and notification

Notify your various teams and other personnel when changes are made to incidents, so they won't miss any critical updates. Escalate incidents by assigning them to new owners and informing the new owners of their assignments. Control when and how incidents are reopened.

### Maintain synchronization with external systems

If you've used playbooks to create tickets in external systems when incidents are created, you can use an update-trigger automation rule to call a playbook that will update those tickets.

## Automation rules execution

Automation rules are run sequentially, according to the order you determine. Each automation rule is executed after the previous one has finished its run. Within an automation rule, all actions are run sequentially in the order in which they are defined.

Playbook actions within an automation rule may be treated differently under some circumstances, according to the following criteria:

| Playbook run time | Automation rule advances to the next action... |
| ----------------- | --------------------------------------------------- |
| Less than a second | Immediately after playbook is completed |
| Less than two minutes | Up to two minutes after playbook began running,<br>but no more than 10 seconds after the playbook is completed |
| More than two minutes | Two minutes after playbook began running,<br>regardless of whether or not it was completed |

### Permissions for automation rules to run playbooks

When a Microsoft Sentinel automation rule runs a playbook, it uses a special Microsoft Sentinel service account specifically authorized for this action. The use of this account (as opposed to your user account) increases the security level of the service.

In order for an automation rule to run a playbook, this account must be granted explicit permissions to the resource group where the playbook resides. At that point, any automation rule will be able to run any playbook in that resource group.

When you're configuring an automation rule and adding a **run playbook** action, a drop-down list of playbooks will appear. Playbooks to which Microsoft Sentinel does not have permissions will show as unavailable ("grayed out"). You can grant Microsoft Sentinel permission to the playbooks' resource groups on the spot by selecting the **Manage playbook permissions** link. To grant those permissions, you'll need **Owner** permissions on those resource groups. [See the full permissions requirements](tutorial-respond-threats-playbook.md#respond-to-incidents).

#### Permissions in a multi-tenant architecture

Automation rules fully support cross-workspace and [multi-tenant deployments](extend-sentinel-across-workspaces-tenants.md#managing-workspaces-across-tenants-using-azure-lighthouse) (in the case of multi-tenant, using [Azure Lighthouse](../lighthouse/index.yml)).

Therefore, if your Microsoft Sentinel deployment uses a multi-tenant architecture, you can have an automation rule in one tenant run a playbook that lives in a different tenant, but permissions for Sentinel to run the playbooks must be defined in the tenant where the playbooks reside, not in the tenant where the automation rules are defined.

In the specific case of a Managed Security Service Provider (MSSP), where a service provider tenant manages a Microsoft Sentinel workspace in a customer tenant, there are two particular scenarios that warrant your attention:

- **An automation rule created in the customer tenant is configured to run a playbook located in the service provider tenant.** 

    This approach is normally used to protect intellectual property in the playbook. Nothing special is required for this scenario to work. When defining a playbook action in your automation rule, and you get to the stage where you grant Microsoft Sentinel permissions on the relevant resource group where the playbook is located (using the **Manage playbook permissions** panel), you'll see the resource groups belonging to the service provider tenant among those you can choose from. [See the whole process outlined here](tutorial-respond-threats-playbook.md#respond-to-incidents).

- **An automation rule created in the customer workspace (while signed into the service provider tenant) is configured to run a playbook located in the customer tenant**.

    This configuration is used when there is no need to protect intellectual property. For this scenario to work, permissions to execute the playbook need to be granted to Microsoft Sentinel in ***both tenants***. In the customer tenant, you grant them in the **Manage playbook permissions** panel, just like in the scenario above. To grant the relevant permissions in the service provider tenant, you need to add an additional Azure Lighthouse delegation that grants access rights to the **Azure Security Insights** app, with the **Microsoft Sentinel Automation Contributor** role, on the resource group where the playbook resides.

    The scenario looks like this:

    :::image type="content" source="./media/automate-incident-handling-with-automation-rules/automation-rule-multi-tenant.png" alt-text="Multi-tenant automation rule architecture":::

    See [our instructions](tutorial-respond-threats-playbook.md#permissions-to-run-playbooks) for setting this up.

## Creating and managing automation rules

You can [create and manage automation rules](create-manage-use-automation-rules.md) from different points in the Microsoft Sentinel experience, depending on your particular need and use case.

- **Automation blade**

    Automation rules can be centrally managed in the **Automation** blade, under the **Automation rules** tab. From there, you can create new automation rules and edit the existing ones. You can also drag automation rules to change the order of execution, and enable or disable them.

    In the **Automation** blade, you see all the rules that are defined on the workspace, along with their status (Enabled/Disabled) and which analytics rules they are applied to.

    When you need an automation rule that will apply to many analytics rules, create it directly in the **Automation** blade.

- **Analytics rule wizard**

    In the **Automated response** tab of the analytics rule wizard, under **Incident automation**, you can view, edit, and create automation rules that apply to the particular analytics rule being created or edited in the wizard.

    You'll notice that when you create an automation rule from here, the **Create new automation rule** panel shows the **analytics rule** condition as unavailable, because this rule is already set to apply only to the analytics rule you're editing in the wizard. All the other configuration options are still available to you.

- **Incidents blade**

    You can also create an automation rule from the **Incidents** blade, in order to respond to a single, recurring incident. This is useful when creating a [suppression rule](#incident-suppression) for [automatically closing "noisy" incidents](false-positives.md).

    You'll notice that when you create an automation rule from here, the **Create new automation rule** panel has populated all the fields with values from the incident. It names the rule the same name as the incident, applies it to the analytics rule that generated the incident, and uses all the available entities in the incident as conditions of the rule. It also suggests a suppression (closing) action by default, and suggests an expiration date for the rule. You can add or remove conditions and actions, and change the expiration date, as you wish.


## Next steps

In this document, you learned about how automation rules can help you to manage your Microsoft Sentinel incidents queue and implement some basic incident-handling automation.

- [Create and use Microsoft Sentinel automation rules to manage incidents](create-manage-use-automation-rules.md).
- To learn more about advanced automation options, see [Automate threat response with playbooks in Microsoft Sentinel](automate-responses-with-playbooks.md).
- For help in implementing playbooks, see [Tutorial: Use playbooks to automate threat responses in Microsoft Sentinel](tutorial-respond-threats-playbook.md).
