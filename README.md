# XM Cloud Style Guide (Next JS)

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
This solution is designed to help developers learn and get started quickly
with Sitecore Containers, the Sitecore Next.js SDK, and Sitecore
Content Serialization.

For simplicity, this solution does not implement Sitecore Helix conventions for
solution architecture. As you begin building your Sitecore solution,
you should review [Sitecore Helix](https://helix.sitecore.net/) and the
[Sitecore Helix Examples](https://sitecore.github.io/Helix.Examples/) for guidance
on implementing a modular solution architecture.

## Configured for Sitecore-based workflow
On first run, the JSS Styleguide sample will be imported via `jss deploy items`, then serialized via `sitecore ser pull`. It is intended that you work directly in Sitecore to define templates and renderings, instead of using the code-first approach. This is also known as "Sitecore-first" JSS workflow. To support this:

* The JSS content workflow is disabled
* Imported items will not be marked as 'protected'
* JSS import warnings in the Content Editor and Experience Editor have been disabled

The code-first Sitecore definitions and routes remain in the JSS project, in case you wish to use them for local development / mocking. You can remove these from `/data` and `/sitecore` if desired. You may also wish to remove the [initial import logic in the `up.ps1` script](./up.ps1#L44).


## Support
The template output as provided is supported by Sitecore. Once changed or amended,
the solution becomes a custom implementation and is subject to limitations as
defined in Sitecore's [scope of support](https://kb.sitecore.net/articles/463549#ScopeOfSupport).

## Prerequisites
* NodeJs 16.x
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

* Scripted invocation of `jss create` and `jss deploy` to initialize a
  Next.js application.
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
* The running `rendering` service uses `next dev` against the mounted Next.js application, and will recompile automatically for any changes you make.
* You can also run the Next.js application directly using `npm` commands within `src\rendering`.
* Debugging of the Next.js application is possible by using the `start:connected` or `start` scripts from the Next.js `package.json`, and the pre-configured *Attach to Process* VS Code launch configuration.
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

After publishing, you can also use this key in order to run the JSS site against.

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
