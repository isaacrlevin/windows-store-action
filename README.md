#
This is a GitHub Action to deploy MSIX Packages to the Windows Store. Most of the code is "borrowed from the already existng [Azure DevOps extension](https://marketplace.visualstudio.com/items?itemName=MS-RDX-MRO.windows-store-publish) of a similar name.

**NOTE**: I am not a very experienced with MSIX and App Bundling in general and I may not be doing the right things here, but it works for [the app I wrote this for](https://github.com/isaacrlevin/PresenceLight).



## Quick start

1. Ensure you meet the [prerequisites](#prerequisites).

2. [Install](https://marketplace.visualstudio.com/items?itemName=MS-RDX-MRO.windows-store-publish) the extension.

3. [Obtain](#obtaining-your-credentials) and [configure](#configuring-your-credentials) your Dev Center credentials.

4. [Add tasks](#task-reference) to your release definitions.

## Prerequisites

1. You must have an Azure AD directory, and you must have [global administrator permission](https://azure.microsoft.com/en-us/documentation/articles/active-directory-assign-admin-roles/) for the directory. You can create a new Azure AD [from Dev Center](https://msdn.microsoft.com/windows/uwp/publish/manage-account-users).

2. You must associate your Azure AD directory with your Dev Center account to obtain the credentials to allow this extension to access your account and perform actions on your behalf.

3. The app you want to publish must already exist: this extension can only publish updates to existing applications. You can [create your app in Dev Center](https://msdn.microsoft.com/windows/uwp/publish/create-your-app-by-reserving-a-name).

4. You must have already [created at least one submission](https://msdn.microsoft.com/windows/uwp/publish/app-submissions) for your app before you can use the Publish task provided by this extension. If you have not created a submission, the task will fail.

5. More information and extra prerequisites specific to the API can be found [here](https://msdn.microsoft.com/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services).

## Obtaining your credentials

Your credentials are comprised of three parts: the Azure **Tenant ID**, the **Client ID** and the **Client secret**.
Follow these steps to obtain them:

1. In Dev Center, go to your **Account settings**, click **Manage users**, and associate your organization's Dev Center account with your organization's Azure AD directory. For detailed instructions, see [Manage account users](https://msdn.microsoft.com/windows/uwp/publish/manage-account-users).

2. In the **Manage users** page, click **Add Azure AD applications**, add the Azure AD application that represents the app or service that you will use to access submissions for your Dev Center account, and assign it the **Manager** role. If this application already exists in your Azure AD directory, you can select it on the **Add Azure AD applications** page to add it to your Dev Center account. Otherwise, you can create a new Azure AD application on the **Add Azure AD applications** page. For more information, see [Add and manage Azure AD applications](https://msdn.microsoft.com/windows/uwp/publish/manage-account-users#add-and-manage-azure-ad-applications).

3. Return to the **Manage users** page, click the name of your Azure AD application to go to the application settings, and copy the **Tenant ID** and **Client ID** values.

4. Click **Add new key**. On the following screen, copy the **Key** value, which corresponds to the **Client secret**. You *will not* be able to access this info again after you leave this page, so make sure to not lose it. For more information, see the information about managing keys in [Add and manage Azure AD applications](https://msdn.microsoft.com/windows/uwp/publish/manage-account-users#add-and-manage-azure-ad-applications).

See more details on how to create a new Azure AD application account in your organizaiton's directory and add it to your Partner Center account [here](https://docs.microsoft.com/en-gb/windows/uwp/publish/add-users-groups-and-azure-ad-applications#create-a-new-azure-ad-application-account-in-your-organizations-directory-and-add-it-to-your-partner-center-account).

## Task reference

### Windows Store - Publish

This action allows you to publish your app on the Store by creating a submission in Dev Center. It has the following parameters:

* Application ID (*string, required*) - The identification for the app. Depending on your selection, this should be either the app ID (visible in the URL of the app's page on Dev Center) or the app primary name (visible on the app's page on Dev Center).

* Delete pending submissions (*bool*) - If checked, will attempt to delete any in-progress submission before starting a new one. Note that only one submission at a time can be pending. Therefore, if this box is not checked and a submission is already pending, the task will fail. Furthermore, submissions created on the Dev Center UI cannot be deleted automatically by the task.

* Package path (*string, optional*) - Path to your app's main package (usually a file in .appx, .xap or .appxbundle format). Minimatch pattern is supported.

* Additional package(s) (*string, optional*) - A list of paths, one per line, of additional packages that your app needs, for example to support multiple platforms. Each individual path supports Minimatch pattern.

* Skip polling (*boolean*) - If checked, will skip polling the submission after committing it to Dev Center. Otherwise, it will keep polling the submission till it gets published (which typically takes around 2 hours). **Warning**: If you set this to true, you will not see errors, if any, that your submission may run into. You will have to manually check the status of your submission in Dev Center.

* Delete Packages (*boolean, optional*) - If checked, will enable deletion of one or more old packages (sorted by version). Checking this box will enable a dropdown "Number of packages to keep" explained in following point. If not checked, will not delete any old package.

* Number of Packages to keep (*int, required*) - Specify number of latest packages (sorted by version) to be kept per unique target device family and target platform. For example, if you have a mix of 3 distinct packages each for Windows 10 desktop, mobile and Windows 8.1 X64 platform (so in total 9 packages), and you specify 2 in this box, then the oldest package in each group will be deleted (total packages after deletion will be 6).

You only have to select the packages you want to update. If you have a package that will not be updated as part of your release, you do not have to specify it.
## Sample

```yml
name: ci

on:
  push:
    branches:
      - 'main'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/download-artifact@v2
      name: Download Release Signed
      with:
        name: ReleaseSigned
        path: "${{ github.workspace }}/ReleaseSigned"

    - uses: isaacrlevin/windows-store-action
      name: Publish to Store
      with:
        tenant-id: ${{ secrets.STORE_TENANT }}
        client-id: ${{ secrets.STORE_CLIENT_ID }}
        client-secret: ${{ secrets.STORE_CLIENT_SECRET }}
        app-id: ${{ secrets.APP_ID }}
        package-path: "${{ github.workspace }}/ReleaseSigned/MyApp.appxupload"


```
