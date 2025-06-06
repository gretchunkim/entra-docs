---
title: Register single-page applications (SPA)
description: Learn how to build a single-page application (app registration)
author: OwenRichards1
manager: CelesteDG
ms.author: owenrichards
ms.custom:
ms.date: 04/29/2024
ms.service: identity-platform

ms.topic: how-to
#Customer intent: As an application developer, I want to know how to write a single-page application by using the Microsoft identity platform.
---

# Single-page application: App registration

[!INCLUDE [applies-to-workforce-only](../external-id/includes/applies-to-workforce-only.md)]

To register a single-page application (SPA) in the Microsoft identity platform, complete the following steps. The registration steps differ between MSAL.js 1.0, which supports the implicit grant flow, and MSAL.js 2.0, which supports the authorization code flow with PKCE. If you are using MSAL.js 1.0, we recommend you migrate to MSAL.js 2.0 to take advantage of the authorization code flow with PKCE which is more secure.

## Create the app registration


For both MSAL.js 1.0- and 2.0-based applications, start by completing the following steps to create the initial app registration.

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com).
1. If you have access to multiple tenants, use the **Settings** icon :::image type="icon" source="media/common/admin-center-settings-icon.png" border="false"::: in the top menu to switch to the tenant in which you want to register the application from the **Directories + subscriptions** menu.
1. Go to **Entra ID** > **App registrations**, select **New registration**.
1. Enter a **Name** for your application. Users of your app might see this name, and you can change it later.
1. Choose the **Supported account types** for the application. Do **NOT** enter a **Redirect URI**. For a description of the different account types, see the [Register an application](quickstart-register-app.md).
1. Select **Register** to create the app registration.

Next, configure the app registration with a **Redirect URI** to specify where the Microsoft identity platform should redirect the client along with any security tokens. Use the steps appropriate for the version of MSAL.js you're using in your application:

- [MSAL.js 2.0 with auth code flow](#redirect-uri-msaljs-20-with-auth-code-flow) (recommended)
- [MSAL.js 1.0 with implicit flow](#redirect-uri-msaljs-10-with-implicit-flow)

Keep in mind that MSAL.js 2.0+ supports the authorization code flow with PKCE which is more secure than the implicit grant flow. If you're using MSAL.js 1.0, consider migrating to MSAL.js 2.0 to take advantage of the authorization code flow with PKCE.

## Redirect URI: [MSAL.js 2.0 with auth code flow](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

Follow these steps to add a redirect URI for an app that uses MSAL.js 2.0 or later. MSAL.js 2.0+ supports the authorization code flow with PKCE and Cross-Origin Resource Sharing (CORS) in response to [browser third party cookie restrictions](reference-third-party-cookies-spas.md). The implicit grant flow isn't supported in MSAL.js 2.0+.

1. In the Microsoft Entra admin center, select the app registration you created in [Create the app registration](#create-the-app-registration).
1. Under **Manage**, select **Authentication** > **Add a platform**.
1. Under **Web applications**, select the **Single-page application** tile.
1. Under **Redirect URIs**, enter a [redirect URI](reply-url.md). Do **NOT** select either checkbox under **Implicit grant and hybrid flows**.
1. Select **Configure** to finish adding the redirect URI.

Your SPA is now registered with a redirect URI. By configuring your redirect URI using the **Single-page application** tile in the **Add a platform** pane, your application registration supports the authorization code flow with PKCE and CORS.

## Redirect URI: [MSAL.js 1.0 with implicit flow](/javascript/api/overview/msal-overview)

Follow these steps to add a redirect URI for a SPA that uses MSAL.js 1.3 or earlier and the implicit grant flow. Applications that use MSAL.js 1.3 or earlier don't support the authorization code flow.

1. In the Microsoft Entra admin center, select the app registration you created in [Create the app registration](#create-the-app-registration).
1. Under **Manage**, select **Authentication** > **Add a platform**.
1. Under **Web applications**, select **Single-page application** tile.
1. Under **Redirect URIs**, enter a [redirect URI](reply-url.md).
1. Enable the **Implicit grant and hybrid flows**:
    - If your application signs in users, select **ID tokens**.
    - If your application also needs to call a protected web API, select **Access tokens**. For more information about these token types, see [ID tokens](id-tokens.md) and [Access tokens](access-tokens.md).
1. Select **Configure** to finish adding the redirect URI.

Your SPA is now registered with a redirect URI. By selecting one or both of **ID tokens** and **Access tokens**, your application registration supports the implicit grant flow.

## Note about authorization flows

By default, an app registration created using the SPA platform configuration enables the authorization code flow. To take advantage of this flow, your application must use MSAL.js 2.0 or later.

SPAs using MSAL.js 1.3 are restricted to the implicit grant flow. Current [OAuth 2.0 best practices](v2-oauth2-auth-code-flow.md) recommend using the authorization code flow rather than the implicit flow for SPAs. Having limited-lifetime refresh tokens also helps your application adapt to [modern browser cookie privacy limitations](reference-third-party-cookies-spas.md), like Safari ITP.

When all your production SPAs represented by an app registration are using MSAL.js 2.0 and the authorization code flow, uncheck the implicit grant settings on the app registration's **Authentication** pane in the Microsoft Entra admin center. If you still have SPAs using MSAL.js 1.x, leave the implicit flow enabled (checked).

## Next steps

Configure your app's code to use the app registration you created in [App's code configuration](scenario-spa-app-configuration.md).
