---
title: Configure an application to trust a managed identity
description: Learn how to configure an application to trust a managed identity in Microsoft Entra ID.
author: cilwerner
manager: CelesteDG
ms.service: entra-workload-id
ms.topic: how-to
ms.date: 12/3/2024
ms.author: cwerner
ms.reviewer: hosamsh
#Customer intent: As an application developer, I want to configure my application to trust a managed identity so that I can access Microsoft Entra protected resources without needing to use or manage application secrets or certificates.
---

# Configure an application to trust a managed identity

This article describes how to configure a Microsoft Entra application to trust a managed identity. You can then exchange the managed identity token for an access token that can access Microsoft Entra protected resources without needing to use or manage App secrets.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- This Azure account must have permissions to [update application credentials](~/identity/role-based-access-control/custom-available-permissions.md#microsoftdirectoryapplicationscredentialsupdate). Any of the following Microsoft Entra roles include the required permissions:
  - [Application Administrator](~/identity/role-based-access-control/permissions-reference.md#application-administrator)
  - [Application Developer](~/identity/role-based-access-control/permissions-reference.md#application-developer)
  - [Cloud Application Administrator](~/identity/role-based-access-control/permissions-reference.md#cloud-application-administrator)
- An understanding of the concepts in [managed identities for Azure resources](/entra/identity/managed-identities-azure-resources/overview).
- [A user-assigned managed identity](/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azp#create-a-user-assigned-managed-identity) assigned to the Azure compute resource (for example, a virtual machine or Azure App Service) that hosts your workload.
- An [app registration](~/identity-platform/quickstart-register-app.md) in Microsoft Entra ID. This app registration must belong to the same tenant as the managed identity 
    - If you need to access resources in another tenant, your app registration must be a multitenant application and provisioned into the other tenant. Learn about [how to add a multitenant app in other tenants](/entra/identity/enterprise-apps/grant-admin-consent).
- The app registration must have access granted to Microsoft Entra protected resources (for example, Azure, Microsoft Graph, Microsoft 365, etc.). This access can be granted through [API permissions](../identity-platform/quickstart-configure-app-access-web-apis.md#add-permissions-to-access-microsoft-graph) or [delegated permissions](../identity-platform/quickstart-configure-app-access-web-apis.md#delegated-permission-to-microsoft-graph).

## Important considerations and restrictions

To create, update, or delete a federated identity credential, the account performing the action must have the [Application Administrator](~/identity/role-based-access-control/permissions-reference.md#application-administrator), [Application Developer](~/identity/role-based-access-control/permissions-reference.md#application-developer), [Cloud Application Administrator](~/identity/role-based-access-control/permissions-reference.md#cloud-application-administrator), or Application Owner role.  The [microsoft.directory/applications/credentials/update permission](~/identity/role-based-access-control/custom-available-permissions.md#microsoftdirectoryapplicationscredentialsupdate) is required to update a federated identity credential.

A maximum of 20 federated identity credentials can be added to an application or user-assigned managed identity.

When you configure a federated identity credential, there are several important pieces of information to provide:

- *issuer*, *subject* are the key pieces of information needed to set up the trust relationship. When the Azure workload requests Microsoft identity platform to exchange the managed identity token for an Entra app access token, the *issuer* and *subject* values of the federated identity credential are checked against the `issuer` and `subject` claims provided in the Managed Identity token. If that validation check passes, Microsoft identity platform issues an access token to the external software workload.
- *issuer* is the URL of the Microsoft Entra tenant's Authority URL in the form `https://login.microsoftonline.com/{tenant}/v2.0`. Both the Microsoft Entra app and managed identity must belong to the same tenant. If the `issuer` claim has leading or trailing whitespace in the value, the token exchange is blocked.   
- `subject`: This is the case-sensitive GUID of the managed identity’s **Object (Principal) ID** assigned to the Azure workload. The managed identity must be in the same tenant as the app registration, even if the target resource is in a different cloud. The Microsoft identity platform will reject the token exchange if the `subject` in the federated identity credential configuration does not exactly match the managed identity's Principal ID.
    > [!IMPORTANT]
    > Only user-assigned managed identities can be used as a federated credential for apps. system-assigned identities aren't supported.
    
- *audiences* specifies the value that appears in the `aud` claim in the managed identity token (Required). The value must be one of the following depending on the target cloud.
    - **Microsoft Entra ID global service**: `api://AzureADTokenExchange`
    - **Microsoft Entra ID for US Government**: `api://AzureADTokenExchangeUSGov`
    - **Microsoft Entra China operated by 21Vianet**: `api://AzureADTokenExchangeChina`
    

  > [!IMPORTANT]  
  > Accessing resources in *another tenant* is supported.
  > Accessing resources in *another cloud* is not supported. Token requests to other clouds will fail.

  > [!IMPORTANT]
  > If you accidentally add incorrect information in the *issuer*, *subject* or *audience* setting the federated identity credential is created successfully without error. The error does not become apparent until the token exchange fails.
    
- *name* is the unique identifier for the federated identity credential. (Required) This field has a character limit of 3-120 characters and must be URL friendly. Alphanumeric, dash, or underscore characters are supported, and the first character must be alphanumeric only.  It's immutable once created.
- *description* is the user-provided description of the federated identity credential (Optional). The description isn't validated or checked by Microsoft Entra ID. This field has a limit of 600 characters.

Wildcard characters aren't supported in any federated identity credential property value.

## Configure a federated identity credential on an application

In this section, you'll configure a federated identity credential on an existing application to trust a managed identity. Use the following tabs to choose how to configure a federated identity credential on an existing application.

### [Microsoft Entra admin center](#tab/microsoft-entra-admin-center)

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com/). Check that you are in the tenant where your application is registered.
1. Browse to **Entra ID** > **App registrations**, and select your application in the main window.
1. Under **Manage**, select **Certificates & secrets**.
1. Select the Federated credentials tab and select **Add credential**.

    :::image type="content" source=".\media\workload-identity-federation-config-app-trust-managed-identity\select-federated-credential.png" alt-text="Screenshot of the certificates and secrets pane of the Microsoft Entra admin center with the federated credentials tab highlighted." ::: 

1. From the **Federated credential scenario** dropdown, select **Managed Identity** and fill in the values according to the following table:

    | Field | Description | Example |
    | --- | --- | --- |
    | Issuer | The OAuth 2.0 / OIDC issuer URL of the Microsoft Entra ID authority that issues the managed identity token. This value is automatically populated with the current Entra tenant issuer. | `https://login.microsoftonline.com/{tenantID}/v2.0` |    
    | Select managed identity | Click on this link to select the managed identity that will act as the federated identity credential. You can only use User-Assigned Managed Identities as a credential. | *msi-webapp1* |
    | Description (Optional) | A user-provided description of the federated identity credential. | *Trust the workloads UAMI as a credential to my App* |
    | Audience | The audience value that must appear in the external token.  | Must be set to one of the following values:<br/> &#8226; **Entra ID Global Service**: *api://AzureADTokenExchange* <br/>&#8226; **Entra ID for US Government**: *api://AzureADTokenExchangeUSGov* <br/>&#8226; **Entra ID China operated by 21Vianet**: *api://AzureADTokenExchangeChina* <br/> |

    :::image type="content" source=".\media\workload-identity-federation-config-app-trust-managed-identity\add-credential.png" alt-text="Screenshot of the credential window in the Microsoft Entra admin center." ::: 

### [Azure CLI](#tab/azure-cli)

Open a terminal in your preferred IDE and run the following command to create a federated identity credential on your app.

```CLI
az ad app federated-credential create --id 00001111-aaaa-2222-bbbb-3333cccc4444 --parameters credential.json
```

The `id` parameter specifies the application ID (object ID). The `parameters` parameter specifies federated identity credential configuration, in JSON format. 

This is an example for the contents of *credential.json*. Replace the `subject` GUID with the Object (principal) ID of the managed identity, and `{tenantID}` with your application's tenant ID. 
The audience value must be set to one of the following values:<br/> &#8226; **Entra ID Global Service**: *api://AzureADTokenExchange* <br/>&#8226; **Entra ID for US Government**: *api://AzureADTokenExchangeUSGov* <br/>&#8226; **Entra ID China operated by 21Vianet**: *api://AzureADTokenExchangeChina* <br/>

```json
{
    "name": "msi-webapp1",
    "issuer": "https://login.microsoftonline.com/{tenantID}/v2.0",
    "subject": "00001111-aaaa-2222-bbbb-3333cccc4444",
    "description": "Trust the workload's UAMI to impersonate the App",
    "audiences": [
        "api://AzureADTokenExchange"
    ]
}
```

### [PowerShell](#tab/powershell)

Open a PowerShell terminal in your preferred IDE and run the following command to create a federated identity credential on your app. Replace the `Subject` GUID with the Object (principal) ID of the managed identity, and `{tenantID}` with your application's tenant ID.

The audience value must be set to one of the following values:<br/> &#8226; **Entra ID Global Service**: *api://AzureADTokenExchange* <br/>&#8226; **Entra ID for US Government**: *api://AzureADTokenExchangeUSGov* <br/>&#8226; **Entra ID China operated by 21Vianet**: *api://AzureADTokenExchangeChina* <br/>

```Powershell
New-AzADAppFederatedCredential -ApplicationObjectId $appObjectId -Audience api://AzureADTokenExchange -Issuer 'https://login.microsoftonline.com/{tenantID}/v2.0' -Name 'MyMsiFic' -Subject '00001111-aaaa-2222-bbbb-3333cccc4444'
```



### [APIs](#tab/api)

Open a terminal in your preferred IDE and run the following command to create a federated identity credential on your app. Set the `subject` value to the Object (principal) ID of the managed identity, and `{tenantID}` with your application's tenant ID.

The audience value must be set to one of the following values:<br/> &#8226; **Entra ID Global Service**: *api://AzureADTokenExchange* <br/>&#8226; **Entra ID for US Government**: *api://AzureADTokenExchangeUSGov* <br/>&#8226; **Entra ID China operated by 21Vianet**: *api://AzureADTokenExchangeChina* <br/>

```bash
az rest --method POST --uri 'https://graph.microsoft.com/applications/{app_registration_id}/federatedIdentityCredentials' --body '{"name":"MyMsiFicTest","issuer":"https://login.microsoftonline.com/{tenantID}/v2.0","subject":"{Managed_Identity_Principal_ID}","description":"Trust the workloads UAMI to impersonate the App","audiences":["api://AzureADTokenExchange"]}'
```

### [Bicep](#tab/bicep)

This example shows how to use Bicep to create a FIC to make your app trust the assigned managed identity. Replace the placeholders with the appropriate values.

```Bicep
extension 'br:mcr.microsoft.com/bicep/extensions/microsoftgraph/v1.0:0.1.8-preview'

param myWorkloadManagedIdentity string = '[MANAGED-IDENTITY-NAME]'
param applicationDisplayName string = '[APPLICATION-DISPLAYNAME]'
param applicationName string = '[APPLICATION-UNIQUE-NAME]'

resource myManagedIdentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' existing = {
  name: myWorkloadManagedIdentity
}

resource myApp 'Microsoft.Graph/applications@v1.0' = {
  displayName: applicationDisplayName
  uniqueName: applicationName

  resource myMsiFic 'federatedIdentityCredentials@v1.0' = {
    name: '${myApp.uniqueName}/msiAsFic'
    description: 'Trust the workloads UAMI to impersonate the App'
    audiences: [
       'api://AzureADTokenExchange'
    ]
    issuer: '${environment().authentication.loginEndpoint}${tenant().tenantId}/v2.0'
    subject: myManagedIdentity.properties.principalId
  }
}
```
---

## Update your application code to request an access token

The following code snippets demonstrate how to acquire a managed identity token and use it as a credential for your Entra application. The samples are valid in both cases where the target resource in the same tenant as the Entra application, or in a different tenant.

### Azure.Identity

This example demonstrates accessing an Azure storage container, but can be adapted to access any resource protected by Microsoft Entra.

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

internal class Program
{
  // This example demonstrates how to access an Azure blob storage account by utilizing the manage identity credential.
  static void Main(string[] args)
  {
    string storageAccountName = "YOUR_STORAGE_ACCOUNT_NAME";
    string containerName = "CONTAINER_NAME";
        
    // The application must be granted access on the target resource
    string appClientId = "YOUR_APP_CLIENT_ID";

    // The tenant where the target resource is created, in this example, the storage account tenant
    // If the resource tenant different from the app tenant, your app needs to be 
    string resourceTenantId = "YOUR_RESOURCE_TENANT_ID";

    // The managed identity which you configured as a Federated Identity Credential (FIC)
    string miClientId = "YOUR_MANAGED_IDENTITY_CLIENT_ID"; 

    // Audience value must be one of the below values depending on the target cloud.
    // Entra ID Global cloud: api://AzureADTokenExchange
    // Entra ID US Government: api://AzureADTokenExchangeUSGov
    // Entra ID China operated by 21Vianet: api://AzureADTokenExchangeChina
    string audience = "api://AzureADTokenExchange";

    // 1. Create an assertion with the managed identity access token, so that it can be exchanged an app token
    var miCredential = new ManagedIdentityCredential(managedIdentityClientId);
    ClientAssertionCredential assertion = new(
        tenantId,
        appClientId,
        async (token) =>
        {
            // fetch Managed Identity token for the specified audience
            var tokenRequestContext = new Azure.Core.TokenRequestContext(new[] { $"{audience}/.default" });
            var accessToken = await miCredential.GetTokenAsync(tokenRequestContext).ConfigureAwait(false);
            return accessToken.Token;
        });

        // 2. The assertion can be used to obtain an App token (taken care of by the SDK)
        var containerClient  = new BlobContainerClient(new Uri($"https://{storageAccountName}.blob.core.windows.net/{containerName}"), assertion);

        await foreach (BlobItem blob in containerClient.GetBlobsAsync())
        {
            // TODO: perform operations with the blobs
            BlobClient blobClient = containerClient.GetBlobClient(blob.Name);
            Console.WriteLine($"Blob name: {blobClient.Name}, uri: {blobClient.Uri}");            
        }
    }
}
```

### Microsoft.Identity.Web

In **Microsoft.Identity.Web**, you can set the `ClientCredentials` section in your *appsettings.json* to use `SignedAssertionFromManagedIdentity` to enable your code use the configured managed identity as a credential:

``` JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "YOUR_APPLICATION_ID",
    "TenantId": "YOUR_TENANT_ID",
    
    "ClientCredentials": [
      {
        "SourceType": "SignedAssertionFromManagedIdentity",
        "ManagedIdentityClientId": "YOUR_USER_ASSIGNED_MANAGED_IDENTITY_CLIENT_ID",
        "TokenExchangeUrl": "api://AzureADTokenExchange"
      }
    ]
  }
}
```

### MSAL (.NET)

In **MSAL**, you can use the [ManagedClientApplication](/entra/msal/dotnet/advanced/managed-identity) class to acquire a Managed Identity token. This token can then be used as a client assertion when constructing a confidential client application.

``` csharp
using Microsoft.Identity.Client;
using Azure.Storage.Blobs;
using Azure.Core;

internal class Program
{
  static async Task Main(string[] args)
  {
      string storageAccountName = "YOUR_STORAGE_ACCOUNT_NAME";
      string containerName = "CONTAINER_NAME";

      string appClientId = "YOUR_APP_CLIENT_ID";
      string resourceTenantId = "YOUR_RESOURCE_TENANT_ID";
      Uri authorityUri = new($"https://login.microsoftonline.com/{resourceTenantId}");
      string miClientId = "YOUR_MI_CLIENT_ID";
      string audience = "api://AzureADTokenExchange";

      // Get mi token to use as assertion
      var miAssertionProvider = async (AssertionRequestOptions _) =>
      {
            var miApplication = ManagedIdentityApplicationBuilder
                .Create(ManagedIdentityId.WithUserAssignedClientId(miClientId))
                .Build();

            var miResult = await miApplication.AcquireTokenForManagedIdentity(audience)
                .ExecuteAsync()
                .ConfigureAwait(false);
            return miResult.AccessToken;
      };

      // Create a confidential client application with the assertion.
      IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(appClientId)
        .WithAuthority(authorityUri, false)
        .WithClientAssertion(miAssertionProvider)
        .WithCacheOptions(CacheOptions.EnableSharedCacheOptions)
        .Build();

        // Get the federated app token for the storage account
        string[] scopes = [$"https://{storageAccountName}.blob.core.windows.net/.default"];
        AuthenticationResult result = await app.AcquireTokenForClient(scopes).ExecuteAsync().ConfigureAwait(false);

        TokenCredential tokenCredential = new AccessTokenCredential(result.AccessToken);
        var client = new BlobContainerClient(
            new Uri($"https://{storageAccountName}.blob.core.windows.net/{containerName}"),
            tokenCredential);

        await foreach (BlobItem blob in containerClient.GetBlobsAsync())
        {
            // TODO: perform operations with the blobs
            BlobClient blobClient = containerClient.GetBlobClient(blob.Name);
            Console.WriteLine($"Blob name: {blobClient.Name}, URI: {blobClient.Uri}");
        }
    }
}
```

## See also

- [Important considerations and restrictions for federated identity credentials](./workload-identity-federation-considerations.md).
