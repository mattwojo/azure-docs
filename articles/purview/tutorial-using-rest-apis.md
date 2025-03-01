---
title: "How to use REST APIs for Microsoft Purview Data Planes"
description: This tutorial describes how to use the Microsoft Purview REST APIs to access the contents of your Microsoft Purview.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.custom: subject-rbac-steps
ms.topic: tutorial
ms.date: 09/17/2021

# Customer intent: I can call the Data plane REST APIs to perform CRUD operations on Microsoft Purview account.
---

# Tutorial: Use the REST APIs

In this tutorial, you learn how to use the Microsoft Purview REST APIs. Anyone who wants to submit data to a Microsoft Purview, include Microsoft Purview as part of an automated process, or build their own user experience on the Microsoft Purview can use the REST APIs to do so.

If you don't have an Azure subscription, [create a free account](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) before you begin.

## Prerequisites

* To get started, you must have an existing Microsoft Purview account. If you don't have a catalog, see the [quickstart for creating a Microsoft Purview account](create-catalog-portal.md).

## Create a service principal (application)

For a REST API client to access the catalog, the client must have a service principal (application), and an identity that the catalog recognizes and is configured to trust. When you make REST API calls to the catalog, they use the service principal's identity.

Customers who have used existing service principals (application IDs) have had a high rate of failure. Therefore, we recommend creating a new service principal for calling APIs.

To create a new service principal:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. From the portal, search for and select **Azure Active Directory**.
1. From the **Azure Active Directory** page, select **App registrations** from the left pane.
1. Select **New registration**.
1. On the **Register an application** page:
    1. Enter a **Name** for the application (the service principal name).
    1. Select **Accounts in this organizational directory only (_&lt;your tenant's name&gt;_ only - Single tenant)**.
    1. For **Redirect URI (optional)**, select **Web** and enter a value. This value doesn't need to be a valid URI.
    1. Select **Register**.
1. On the new service principal page, copy the values of the **Display name** and the **Application (client) ID** to save for later.

   The application ID is the `client_id` value in the sample code.

To use the service principal (application), you need to get
its password. Here's how:

1. From the Azure portal, search for and select **Azure Active Directory**, and then select **App registrations** from the left pane.
1. Select your service principal (application) from the list.
1. Select **Certificates & secrets** from the left pane.
1. Select **New client secret**.
1. On the **Add a client secret** page, enter a **Description**, select an expiration time under **Expires**, and then select **Add**.

   On the **Client secrets** page, the string in the **Value** column of your new secret is your password. Save this value.

   :::image type="content" source="./media/tutorial-using-rest-apis/client-secret.png" alt-text="Screenshot showing a client secret.":::

## Set up authentication using service principal

Once service principal is created, you need to assign Data plane roles of your purview account to the service principal created above. The below steps need to be followed to assign role to establish trust between the service principal and purview account.

1. Navigate to your [Microsoft Purview Studio](https://web.purview.azure.com/resource/).
1. Select the Data Map in the left menu.
1. Select Collections.
1. Select the root collection in the collections menu. This will be the top collection in the list, and will have the same name as your Microsoft Purview account.

    >[!NOTE] 
    >You can also assign your service principal permission to any sub-collections, instead of the root collection. However, all APIs will be scoped to that collection (and sub-collections that inherit permissions), and users trying to call the API for another collection will get errors.

1. Select **Access control (IAM)**.

1. Select **Add** > **Add role assignment** to open the **Add role assignment** page.

    ![Screenshot that shows Add role assignment page in Azure portal.](../../includes/role-based-access-control/media/add-role-assignment-page.png)

1. Select the **Role** tab.

1. Assign the following roles to the service principal created previously to access various data planes in Microsoft Purview. For detailed steps, see [Assign Azure roles using the Azure portal](../role-based-access-control/role-assignments-portal.md).

    * Data Curator role to access Catalog Data plane.
    * Data Source Administrator role to access Scanning Data plane.
    * Collection Admin role to access Account Data Plane and Metadata policy Data Plane.

    > [!Note]
    > Only members of the Collection Admin role can assign data plane roles in Microsoft Purview. For more information about Microsoft Purview roles, see [Access Control in Microsoft Purview](./catalog-permissions.md).

## Get token
You can send a POST request to the following URL to get access token.

https://login.microsoftonline.com/{your-tenant-id}/oauth2/token

The following parameters need to be passed to the above URL.

- **client_id**:  client ID of the application registered in Azure Active directory and is assigned to a data plane role for the Microsoft Purview account.
- **client_secret**: client secret created for the above application.
- **grant_type**: This should be ‘client_credentials’.
- **resource**: This should be ‘https://purview.azure.net’
 
Sample response token:

```json
    {
        "token_type": "Bearer",
        "expires_in": "86399",
        "ext_expires_in": "86399",
        "expires_on": "1621038348",
        "not_before": "1620951648",
        "resource": "https://purview.azure.net",
        "access_token": "<<access token>>"
    }
```

Use the access token above to call the Data plane APIs.

## Next steps

> [!div class="nextstepaction"]
> [Manage data sources](manage-data-sources.md)
> [Microsoft Purview Data Plane REST APIs](/rest/api/purview/)
