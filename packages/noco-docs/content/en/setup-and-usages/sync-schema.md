---
title: 'Sync Schema'
description: 'Schema changes made to database from outside NocoDB GUI can be synced'
position: 610
category: 'Product'
menuTitle: 'Sync Schema'
---

## How to sync schema changes to NocoDB

NocoDB allows you to sync schema changes if you have made changes outside NocoDB GUI. However, it has to be noted then you will have to bring your own schema migrations for moving from one environment to other.

Below are the steps to sync schema changes.

### 1. From the `Project menu`, click `Team & Settings`

<img width="390" alt="image" src="https://user-images.githubusercontent.com/35857179/189115289-07657c15-deab-435f-b0f9-2948007f8c65.png">

### 2. Click `Project Metadata` under SETTINGS, access `Metadata` tab

![Screenshot 2022-09-13 at 10 37 17 AM](https://user-images.githubusercontent.com/86527202/189814111-56036958-c6e4-4560-af1e-9443380db080.png)

### 3. Changes carried outside GUI, identified by NocoDB are listed under `Sync state`
- If changes made to the database are not visible, click `Reload`
- Identified schema changes are identified for each table in `red`
- `Sync now` button gets activated, if Schema changes are identified by NocoDB

![Screenshot 2022-09-13 at 10 42 12 AM](https://user-images.githubusercontent.com/86527202/189814648-ca28f28d-b0ed-4652-a5da-e6472bfd9407.png)
<!-- ![image](https://user-images.githubusercontent.com/35857179/161957119-f66f22ad-9d37-45ed-84ca-35c99726078c.png) -->

### 4. Click `Sync Now` to complete Schema sync procedure

#### Notes

1. Column rename operation will be treated like `column delete` & `column create` operation.
2. Only table schema changes identified will be listed under `Sync state`. Any changes to records are automatically synchronized.





