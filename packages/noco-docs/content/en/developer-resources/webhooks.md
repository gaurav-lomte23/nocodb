---
title: "Webhooks"
description: "Webhooks allows user to trigger on certain operations on following database operations"
position: 1500
category: "Developer Resources"
menuTitle: "Webhooks"
---

## Overview

Some types of notifications can be triggered by a webhook after a particular event.

### Open `View menu`, click on `Webhooks`
  
![Screenshot 2022-09-14 at 10 32 13 AM](https://user-images.githubusercontent.com/86527202/190064555-77d2444e-250e-4c26-bf65-4bccde166c25.png)

<!-- ![Screenshot 2022-02-22 at 11 16 18 AM](https://user-images.githubusercontent.com/86527202/155085373-f9b438ed-98c3-4fb1-9209-1bb52736a35d.png) -->

### Click `Add new webhook`
  
![Screenshot 2022-09-14 at 10 33 15 AM](https://user-images.githubusercontent.com/86527202/190064639-c51038bc-cfd0-4f5a-aece-3bade55ae994.png)

<!-- ![image](https://user-images.githubusercontent.com/35857179/166660074-0a896ec9-9cd8-403e-a713-61c2cefbae28.png) -->

### Configure Webhook
  - General configurations
    - Webhook Name
    - Webhook Trigger
    - Webhook Type
  - Webhook Type specific configuration : additional configuration details depending on webhook type selected
  - Webhook Conditional Trigger
    - Only records meeting the criteria will trigger webhook   
  
![Screenshot 2022-09-14 at 10 35 39 AM](https://user-images.githubusercontent.com/86527202/190064668-37245025-81f6-491c-b639-83c8fd131bc3.png)


<!-- ![image](https://user-images.githubusercontent.com/35857179/166660248-a3c81a34-4334-48c2-846a-65759d761559.png) -->


## Triggers

Webhooks allows user to trigger on certain operations on following database operations

-   AFTER INSERT
-   AFTER UPDATE
-   AFTER DELETE

The triggers will trigger asynchronously without blocking the actual operation.

### Applications/services

| Trigger         | Details                                        |
| --------------- | ---------------------------------------------- |
| Email           | Send email to certain email addresses          |
| Slack           | Notify via Slack channel                       |
| Microsoft Teams | Notify via Microsoft Teams channel             |
| Discord         | Notify via Discord channel                     |
| Mattermost      | Notify via Mattermost channel                  |
| Twilio          | Send SMS to certain mobile numbers             |
| Whatsapp Twilio | Send Whatsapp messages to numbers using Twilio |
| URL             | Invoke an HTTP API                             |

  
## Accessing Data: Handlebars

The current row data and other details will be available in the hooks payload so the user can use [handlebar syntax](https://handlebarsjs.com/guide/#simple-expressions) to use data.

> We are using [Handlebars](https://handlebarsjs.com/) library to parse the payload internally.

### Example

For a table with column names (id, title, created_at, updated_at).  
For INSERT/ UPDATE based triggers, use following handlebars to access corresponding **data** fields.

-   {{ **data**.id }}
-   {{ **data**.title }}
-   {{ **data**.created_at }}
-   {{ **data**.updated_at }}  
  
Note that, for Update trigger - all the fields in the ROW will be accessible, not just the field updated.
For DELETE based triggers, **only** {{ data.id }} is accessible representing ID of the column deleted.
  
### JSON format

Use {{ json data }} to dump complete data & user information available in JSON format

### Additional references:

[Handlebar Guide](https://handlebarsjs.com/guide/).

# Application Guide

## Discord

### 1. Create WebHook

-   On Discord, open your Server Settings and head into the Integrations tab:
-   Click the "Create Webhook" button to create a new webhook!

![Screenshot 2022-02-22 at 1 21 59 PM](https://user-images.githubusercontent.com/86527202/155087088-8f9fd762-9ff9-41a6-aed4-0f22add77fe6.png)

-   Choose channel to which this webhook will post to.
-   Copy webhook URL

![Screenshot 2022-02-22 at 1 23 18 PM](https://user-images.githubusercontent.com/86527202/155087126-c2cdd7b2-518a-46a5-82a5-aa90fe51a709.png)

(Sample webhook URL: https://discord.com/api/webhooks/945558283756908644/GNUtiGuzfOky6wZ4ce30XuXc1sbPK3Od7EC-4t6hihh5Fovv6oU9OsdT6mGuoL1QlTzj).  
Detailed procedure for discord webhook described [here](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks).

### 2. Install Plugin

-   Open 'App Store' (under Settings), hover over Discord tile. Click 'Install'.
  
![Screenshot 2022-09-14 at 10 47 59 AM](https://user-images.githubusercontent.com/86527202/190066333-04bab4eb-f114-48e5-b3f9-6327cadb1ca7.png)
<!-- ![Screenshot 2022-02-22 at 11 30 36 AM](https://user-images.githubusercontent.com/86527202/155085257-5bdde1d9-d7b5-471d-bf44-3c3920e7b853.png) -->

-   Provide a name of your choice (not to be confused with Discord Channel name).
-   Paste Discord Webhook URL copied from Step (1.) above.
  
![Screenshot 2022-09-14 at 10 52 14 AM](https://user-images.githubusercontent.com/86527202/190066365-90e3136b-db24-4514-aa89-c1fb371b33ee.png)
<!-- ![Screenshot 2022-02-22 at 11 31 21 AM](https://user-images.githubusercontent.com/86527202/155085287-f5e45aab-fd33-4138-a7a9-6eddc6dc140b.png) -->

### 3. Configure 

-   Open project and choose a table.
-   Click 'More' > 'Webhooks'.
-   Click 'Create webhook'
-   Configure webhook
    -   **Title**: Name of your choice to identify this Webhook.
    -   **Event**: Trigger event. Choose between.
        -   After Insert: Trigger event for new ROW insertion.
        -   After Update: Trigger event for existing ROW updation.
        -   After Delete: Trigger event for ROW deletion
    -   **On Condition**: [Optional] Enable if you wish to associate additional condition/constraint with the trigger configured above.
    -   **Notification**: Select 'Discord'.
    -   **Select Discord Channels**: Select from the drop down list, channel name configured in Step (2). Please click on 'Reload' if drop down list is empty.
    -   **Body**: Message to be posted over Discord channel, via webhooks on trigger of configured event.
        -   Body can contain plain text &
        -   Handlebars {{ }}


## Slack
### 1. Create WebHook
-   Details to create slack webhook are captured [here](https://api.slack.com/messaging/webhooks)

### 2. Install Plugin
- Procedure remain same as listed for DISCORD channel configuration above

### 3. Configure Webhook
- Procedure remain same as listed for DISCORD channel configuration above
  
  
## Microsoft Teams
### 1. Create WebHook

-   On Teams, open your channel, click on three-dots menu (far right) and select 'Connectors'
  
<img width="319" alt="154971352-6912d53b-cf71-4edd-a319-1c85be85f0c5" src="https://user-images.githubusercontent.com/86527202/155095745-91abd708-834f-4f0e-a33c-ac362e60af0f.png">

  
-   Select incoming webhook & click 'Configure'
  
<img width="442" alt="154971434-0ced97f7-205a-4e2e-8f88-17092cb7771a" src="https://user-images.githubusercontent.com/86527202/155095741-b23ad6b2-1276-46e3-8ada-0d0a871115bb.png">
  
-   Create webhook, Copy webhook URL
  
![154971683-db16be7f-4f07-4447-8f2e-ac50e133bef8](https://user-images.githubusercontent.com/86527202/155095733-c339a914-5d78-408c-8f1e-9cd75a7783e8.png)


### 2. Install Plugin

-   Open 'App Store' (under Settings), hover over 'Microsoft Teams' tile. Click 'Install'.
  
![Screenshot 2022-09-14 at 10 53 22 AM](https://user-images.githubusercontent.com/86527202/190066409-03311add-3b36-4521-acf7-dba5ffdef3fb.png)

<!-- ![Screenshot 2022-02-22 at 7 32 52 PM](https://user-images.githubusercontent.com/86527202/155148122-60844b42-7d2a-4c0f-9778-a5bc4f9c0107.png) -->


-   Provide a name of your choice (not to be confused with Teams Channel name).
-   Paste MS Teams Webhook URL copied from Step (1.) above.
  
![Screenshot 2022-09-14 at 10 53 31 AM](https://user-images.githubusercontent.com/86527202/190066430-838eaa69-ac2c-49ce-a0eb-d84c97964f8b.png)

<!-- <img width="414" alt="154971222-7fe2c25a-d8c6-46b0-ba1e-a05ff1cf6537" src="https://user-images.githubusercontent.com/86527202/155095720-ff1c052c-a4a7-4c10-8f30-d779dac336f3.png"> -->

### 3. Configure 

- Open project and choose a table.
- Click 'More' > 'Webhooks'.
- Click 'Create webhook'
- Configure webhook
    -   **Title**: Name of your choice to identify this Webhook.
    -   **Event**: Trigger event. Choose between.
        -   After Insert: Trigger event for new ROW insertion.
        -   After Update: Trigger event for existing ROW updation.
        -   After Delete: Trigger event for ROW deletion
    -   **On Condition**: [Optional] Enable if you wish to associate additional condition/constraint with the trigger configured above.
    -   **Notification**: Select 'Microsoft Teams'.
    -   **Select Teams Channels**: Select from the drop down list, channel name configured in Step (2). Please click on 'Reload' if drop down list is empty.
    -   **Body**: Message to be posted over Teams channel, via webhooks on trigger of configured event.
        -   Body can contain plain text &
        -   Handlebars {{ }}
