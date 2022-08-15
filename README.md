# CM Modules Guide

## QUICK START

1. In an ADMIN terminal:

    ```ps1
    .\init.ps1 -InitEnv -LicenseXmlPath "C:\path\to\license.xml" -AdminPassword "DesiredAdminPassword"
    ```

2. Restart your terminal and run:

    ```ps1
    .\up.ps1
    ```

3. Follow the instructions to [deploy to XM Cloud](#deploy-to-xmcloud)

4. Create Edge token and [query from edge](#query-edge)

*** 

## About this Solution
This solution is used for managing CM Modules like RAZL, Data Exchange and CMP

## Support
The template output as provided is supported by Sitecore. Once changed or amended,
the solution becomes a custom implementation and is subject to limitations as
defined in Sitecore's [scope of support](https://kb.sitecore.net/articles/463549#ScopeOfSupport).

## Prerequisites
* .NET 6.0 SDK
* .NET Framework 4.8 SDK
* Visual Studio 2019
* Docker for Windows, with Windows Containers enabled

See Sitecore Containers documentation for more information on system requirements.

## What's Included
* A `docker-compose` environment.
  > The included `docker-compose.yml` is a stock environment from the Sitecore
  > Container Support Package. All changes/additions for this solution are included
  > in the `docker-compose.override.yml`.

* Sitecore Content Serialization configuration.
* An MSBuild project for deploying configuration and code into
  the Sitecore Content Management role. (see `src\platform`).

## Running this Solution
1. If your local IIS is listening on port 443, you'll need to stop it.
   > This requires an elevated PowerShell or command prompt.
   ```
   iisreset /stop
   ```

1. Before you can run the solution, you will need to prepare the following
   for the Sitecore container environment:
   * Required environment variable values in `.env` for the Sitecore instance
     * (Can be done once, then checked into source control.)

   See Sitecore Containers documentation for more information on these
   preparation steps. The provided `init.ps1` will take care of them,
   but **you should review its contents before running.**

   > You must use an elevated/Administrator Windows PowerShell 5.1 prompt for
   > this command, PowerShell 7 is not supported at this time.

    ```ps1
    .\init.ps1 -InitEnv -LicenseXmlPath "C:\path\to\license.xml" -AdminPassword "DesiredAdminPassword"
    ```

    If you check your `.env` into source control, other developers
    can prepare a certificate and hosts file entries by simply running:

    ```ps1
    .\init.ps1
    ```

    > Out of the box, this example does not include `.env` in the `.gitignore`.
    > Individual users may override values using process or system environment variables.
    > This file does contain passwords that would provide access to the running containers
    > in the developer's environment. If your Sitecore solution and/or its data are sensitive,
    > you may want to exclude these from source control and provide another
    > means of centrally configuring the information within.

1. If this is your first time using `mkcert` with NodeJs, you will
   need to set the `NODE_EXTRA_CA_CERTS` environment variable. This variable
   must be set in your user or system environment variables. The `init.ps1`
   script will provide instructions on how to do this.
    * Be sure to restart your terminal or VS Code for the environment variable
      to take effect.

1. After completing this environment preparation, run the startup script
   from the solution root:
    ```ps1
    .\up.ps1
    ```

1. When prompted, log into Sitecore via your browser, and
   accept the device authorization.
    * To log in via client credentials flow, set the environment variable SITECORE_FedAuth_dot_Auth0_dot_ClientCredentialsLogin to "true" and update values of the correspond environment variables (SITECORE_FedAuth_dot_Auth0_dot_Domain, SITECORE_FedAuth_dot_Auth0_dot_ClientCredentialsLogin_ClientId, SITECORE_FedAuth_dot_Auth0_dot_ClientCredentialsLogin_ClientSecret, SITECORE_FedAuth_dot_Auth0_dot_ClientCredentialsLogin_Audience, SITECORE_XmCloud_dot_OrganizationId)

1. Wait for the startup script to open browser tabs for the rendered site
   and Sitecore Launchpad.

## Using the Solution
* A Visual Studio / MSBuild publish of the `Platform` project will update the running `cm` service.
* Review README's found in the projects and throughout the solution for additional information.

<a name="deploy-to-xmcloud">
## Deploy your environment to XM Cloud
</a>

* Login to XM Cloud
```
dotnet sitecore cloud login
```

* Create a Project
```
dotnet sitecore cloud project create -n {PROJECT_NAME}
```

* Create an Environment
```
dotnet sitecore cloud environment create --project-id {PROJECT_ID} -n {ENVIRONMENT_NAME}
```

* NOTE THE ENVIRONMENT ID

* Provision and Deploy the Environment with the Starter Kit source code
```
dotnet sitecore cloud deployment create --environment-id {ENVIRONMENT_ID} --upload
```

* Connect to the environment
```
dotnet sitecore cloud environment connect --environment-id {ENVIRONMENT_ID}
```

* Publish the edge content

```
$connectionName = (dotnet sitecore cloud environment info -id {ENVIRONMENT_ID} --json | ConvertFrom-Json).name
dotnet sitecore publish --pt Edge -n $connectionName
```

<a name="query-edge">
## Create an Edge Token and Query from Edge
</a>

Running the following script with the environment id from the previous steps will create an Edge access token and launch the GraphQL Playground so that you can query content.

```ps1
.\New-EdgeToken.ps1 -EnvironmentId {ENVIRONMENT_ID}
```

Example GraphQL Query:

```graphql
query {
  item(path:"/sitecore/content", language:"en") {
    id
  }
  layout(language:"en", routePath:"/",site:"xmcloudpreview"){
    item {
      rendered
    }
  }
  site {
    siteInfoCollection{
      name
    }
  }
}
```

## Rebuild Indexes

After running `.\up.ps1` for the first time, or if you ever run `\docker\clean.ps1`, you will need to [rebuild the search indexes](https://doc.sitecore.com/developers/101/platform-administration-and-architecture/en/rebuild-search-indexes.html).  You can rebuild indexes by running:

```
dotnet sitecore index rebuild
```

You should now be able to view the XmCloudPreview site at https://www.xmcloudpreview.localhost.

## Stop Sitecore

When you're done, stop and remove the containers using the following command.

```
docker-compose down
```

or

```ps1
.\down.ps1
```
