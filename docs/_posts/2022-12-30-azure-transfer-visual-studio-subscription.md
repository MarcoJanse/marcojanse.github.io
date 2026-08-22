---
layout: post
title: Transfer your Visual Studio Enterprise subscription to another Azure tenant
categories: [azure]
---

- [Introduction](#introduction)
- [Best practices](#best-practices)
- [Prerequisites](#prerequisites)
- [Steps](#steps)
  - [Invite/create guest account in source tenant](#invitecreate-guest-account-in-source-tenant)
  - [Accept the invitation](#accept-the-invitation)
  - [Validate Guest user and assign subscription permissions](#validate-guest-user-and-assign-subscription-permissions)
  - [Allow the subscription to leave and enter the tenant (new since May 2026)](#allow-the-subscription-to-leave-and-enter-the-tenant-new-since-may-2026)
  - [Move the subscription to the target tenant](#move-the-subscription-to-the-target-tenant)
  - [Remove Guest account from the source tenant](#remove-guest-account-from-the-source-tenant)
- [Closing notes](#closing-notes)

> **Update (August 2026):** Microsoft changed the default behavior of Azure subscription transfers. Since May 1, 2026, every tenant's "subscription transfer policy" defaults to blocking both inbound and outbound transfers, so the steps below now include an extra step to explicitly allow the move. If you followed this guide before and it "just worked," that's why it might not anymore. See the new [Allow the subscription to leave and enter the tenant](#allow-the-subscription-to-leave-and-enter-the-tenant-new-since-may-2026) step.

## Introduction

Chances are that you are reading this because your company gave you a Visual Studio subscription and you have activated your free monthly Azure credits while still being logged in with your company's Entra ID account, resulting in associating your Visual Studio Enterprise MPN subscription to your company's Entra ID tenant, which usually has a couple of drawbacks:

- You probably won't have owner/administrator permissions in the Entra ID tenant
- There are a lot of existing resources in your company's tenant, which makes it far from the ideal Azure playground

So for example, creating your own Azure Landing Zone will not be possible as you will need full access to the tenant and all subscriptions.

## Best practices

To use your monthly Azure credits to the fullest, you should link this subscription to a new Entra ID tenant, or greenfield tenant, for which you have full administrative access and can build and test from scratch without interfering with any existing resources or dealing with any limitations.

Fortunately, there is a way to move your Azure subscription to another Entra ID tenant.

## Prerequisites

- First of all, you need to have an Entra ID account in your company's tenant. This is the source tenant.
- Secondly, you need to have a second Entra ID tenant/Azure environment that you have full control over (Global Administrator role). If you don't have one, I suggest you get yourself a [Microsoft 365 Business Premium Trial for one month](https://go.microsoft.com/fwlink/p/?LinkID=2102309&clcid=0x409&culture=en-us&country=US) or an [Azure free trial](https://azure.microsoft.com/en-us/offers/ms-azr-0044p/) to get yourself a nice clean tenant and some additional Azure credits and/or some Microsoft 365 functionality to test.
- You need to identify which subscription is yours. If there are multiple MPN subscriptions, you should be able to find yours via [Portal settings \| Directory + subscriptions](https://portal.azure.com/#settings/directory)
- Make sure there are no resources linked to this subscription. (you can move some resources, with your subscription, but it's generally better for these kind of subscriptions to just start over with deploying resources in the new tenant)
- You will need account operator or owner permissions on the Visual Studio Enterprise subscription in the source tenant.
- You need to be able to invite guests in your company's Entra ID tenant. You can find this under `User settings` - `External collaboration settings`

<br>

![External collaboration settings](/assets/images/post_2022-12_azure-azuread-externalcollaboration-settings.png)

<br>

- In my company's tenant, I had these permissions as I was a member of an administrative role and the Guest invite settings were set to `Member users and users assigned to specific admin roles can invite guest users including guests with member permissions`. If you don't have these permissions, someone with administrative privileges will have to do the invitation.
- **New:** Someone with the Global Administrator role in your company's (source) tenant will need to be available to adjust the subscription transfer policy — see the new step below. If you don't hold that role yourself in the source tenant (most people won't), you'll need to loop in your IT/identity team for that one action.

With all these requirements covered, let's get started!

## Steps

### Invite/create guest account in source tenant

- In your source tenant go to the [Entra ID portal](https://entra.cmd.ms)
- Under Users - New Users, select the `Invite External User` option
  - Depending on the template you will only need to enter the Email address of the administrator of the target tenant or fill in some more details.
- After this, an Email should be sent to the target tenant's administrator account.

### Accept the invitation

- Open the Email of the user/administrator from the other tenant and you should have an Email from [invites@microsoft.com](mailto:invites@microsoft.com).
- Using the link from the Email accept the invitation

### Validate Guest user and assign subscription permissions

- Back in the source tenant, check the Entra ID users. There should now be a new guest user from your target tenant
- In the Azure portal, go to [Subscriptions blade](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) and select the Visual Studio Enterprise subscription you want to move.
- Go to Access Control (IAM)
  - Select Role Assignments and click **+Add** and `Add Role Assignment`
  - Select `Owner` as the role
  - Under, members click the `Select Members` and lookup up the guest account you invited in the previous step
  - Click Next and Finish
  - Your Guest user should now be able to manage this subscription

<br>

![Add role assignment](/assets/images/post_2022-12_azure-IAM-add-role-assignment.png)

<br>

### Allow the subscription to leave and enter the tenant (new since May 2026)

This is a step that didn't exist when I originally wrote this article. Since **May 1, 2026**, every Entra ID/Entra tenant has two subscription transfer policies that default to **"Allow no users"**:

- **Subscriptions leaving the directory** — controls outbound transfers, evaluated against the source (company) tenant
- **Subscriptions entering the directory** — controls inbound transfers, evaluated against the target (your own) tenant

With the new default, if you skip this step and go straight to "Change directory" in the next section, you'll hit this error even though you're an Owner on the subscription:

> Your current settings do not allow you to transfer this subscription. Contact your Global Administrator for access.

To fix it, both sides need to explicitly allow the transfer:

**In the source tenant** (your company's tenant) — this needs an actual Global Administrator of that tenant, not just the guest Owner account:

1. Sign in to the [Azure portal](https://portal.azure.com) and go to Microsoft Entra ID → Properties. If "Access management for Azure resources" isn't already enabled, turn it on (this is the [elevated access](https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin) step — it's what lets a Global Admin manage subscription policies at all).
2. Go to the [Subscriptions blade](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) and click **Manage Policies**.
3. Under **Subscriptions leaving the directory**, either set it to "Allow all users," or — the more conservative option, and the one I'd recommend for a company tenant — add the guest account's UPN (it'll look like `you_yourtenant.onmicrosoft.com#EXT#@companytenant.onmicrosoft.com`) to the **exempted users** list instead of opening it up broadly.
4. Save.

**In the target tenant** (your own lab tenant) — you're already Global Admin here, so you can self-serve:

1. Repeat the same elevated-access check under Microsoft Entra ID → Properties, if needed.
2. Go to Subscriptions → **Manage Policies** and confirm **Subscriptions entering the directory** allows your account (either "Allow all users," since it's your own tenant, or add yourself to the exempted list).

<br>

![Subscription Policy Settings](/assets/images/post_2022-12_azure-subscriptionpolicy-settings.png)

<br>

Since this policy change is quite recent, don't be surprised if the person you ask in your company's IT team hasn't run into it yet either — feel free to point them at [Microsoft's docs on subscription transfer policies](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/manage-azure-subscription-policy).

### Move the subscription to the target tenant

- Login to Azure with the guest user account on the source tenant
  - You might need to do this by logging in to your own tenant and switch directory by using the [Portal settings \| Directory + subscriptions](https://portal.azure.com/#settings/directory)
  - There, switch to the source tenant
- Go to [Subscriptions blade](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) and select the Visual Studio Enterprise subscription you want to move.
- This time, choose `Change directory`

<br>

![Subscription options](/assets/images/post_2022-12_azure-subsciption-options-change-directory.png)

<br>

- You should be able to select the source and target directory, as you have access to both and subscription owner permissions
- Initiate the move after reading and selecting the disclaimer and click on `Change`

The move will take some time to reflect on both subscriptions. Give it about 20 minutes and click refresh and repeat until you no longer see the subscription in the source tenant and are able to see the subscription in the target tenant. Do not proceed to the last step until this is ok.

### Remove Guest account from the source tenant

Finally, we can clean up the guest account in the source tenant.

- The easiest way to do this is by using your own tenant account and going to [https://myaccount.microsoft.com/organizations](https://myaccount.microsoft.com/organizations)
- Here you should be able to select the source tenant and leave.

## Closing notes

I hope this article will help you to migrate your Visual Studio Enterprise (or any) Azure subscription.

If you want to migrate a subscription with resources to a different tenant, have a look at this article: [Transfer an Azure subscription to a different Entra ID directory \| Microsoft Learn](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription)

If you hit the "current settings do not allow you to transfer this subscription" error added in 2026, these are the official references that explain what's going on:

- [Manage Azure subscription policies](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/manage-azure-subscription-policy)
- [Elevate access to manage all Azure subscriptions and management groups](https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin)
- [How to change the Entra directory of your Azure subscription](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/subscription-change-directory)
