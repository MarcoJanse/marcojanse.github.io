---
layout: post
title: Set specific database role permissions Azure SQL
categories: powershell azure sql rbac
---

## Introduction

I had to set permissions on multiple databases on an Azure SQL server. 
I decided to create a function for this that can set specific database role permissions on all or one SQL database within an Azure SQL server.

![Azure SQL](../assets/images/post_2024-04_azureSql.png){:width="50%"}

The script uses sqlcmd to connect to the Azure SQL server and configure the permissions. So apart from PowerShell, you will need to have the Az.Accounts PowerShell module and the SqlServer PowerShell module installed. 
Or you can automate it even further by using an Azure DevOps pipeline or GitHub action that first installs PowerShell and the required modules.

Of course you need to either connect using the Sql administrator account or the Microsoft Entra admin for Azure Sql. (The Entra admin can also be a security group in Entra Id)

SqlCmd is very picky about quotes, especially if your Entra ID group has spaces, so the script does some replacing of single quotes (') with two single quotes ('').

## The script

Without further ado, here's the script:

{% gist MarcoJanse/f78534f8d22a9b215a476a122ac5e6aa %}

## Closing notes

I hope someone will make good use of this script or make it even better.

## References

- [Authorize database access to SQL Database, SQL Managed Instance, and Azure Synapse Analytics \| Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage?view=azuresql)