---
layout: default
title: Security Roles
nav_order: 4
parent: Alerting
has_children: false
---

# Security roles

If you use the Security plugin alongside alerting, you might want to limit certain users to certain permissions. For example, you might want some users to only be able to view and acknowledge alerts, while others can create monitors and destinations. This page contains some sample roles for common use cases.

Mix and match these roles to give users the permissions they need to use the alerting feature.

Monitors run as the `admin` user, which means that monitors can query all documents in all indices and do not consider the roles of the user who created the monitor. Please keep this fact in mind while working with sensitive data.
{: .warning }


## View and acknowledge alerts

1. In Kibana, choose **Security**, **Roles**.
1. Create a new security role named `alerting_alerts`.
1. In the **Index Permissions** tab, choose **Add new index and document type**.
1. Specify the `.opendistro-alerting-alerts` index and `*` document type and **Save**.
1. Choose the `CRUD` action group and **Save role definition**.
1. Choose **Role Mappings**.
1. Map the `alerting_acknowledge` role to the desired users or backend roles.


## Create, update, and delete monitors and destinations

1. In Kibana, choose **Security**, **Roles**.
1. Create a new security role named `alerting_monitors`.
1. In the **Index Permissions** tab, choose **Add new index and document type**.
1. Specify the `.opendistro-alerting-config` index and `*` document type and **Save**.
1. Choose the `CRUD` action group and **Save role definition**.
1. Choose **Role Mappings**.
1. Map the `alerting_monitors` role to the desired users or backend roles.


## Read-only

1. In Kibana, choose **Security**, **Roles**.
1. Create a new security role named `alerting_read_only`.
1. In the **Index Permissions** tab, choose **Add new index and document type**.
1. Specify the `.opendistro-alerting-alerts` index and `*` document type and **Save**.
1. Choose the `READ` action group and **Add new index and document type**.
1. Specify the `.opendistro-alerting-config` index and `*` document type and **Save**.
1. Choose the `READ` action group and **Save role definition**.
1. Choose **Role Mappings**.
1. Map the `alerting_read_only` role to the desired users or backend roles.
