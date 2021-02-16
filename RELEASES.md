# Releasing new versions of a package

Outlines the general steps for publishing a new version of a package.

## General

Regardless of the repository's technical details for building a release build, there are steps that are technology agnostic.

1. Create release build artifacts

    - Update project to next version number
    - Create new release build using technology specific steps (see below)
    - Publish new version to respective registry (ie. npm, nuget, etc...) (see below)
    - Commit any build artifacts in git (`git commit -m "Rev'd to version 0.0.x"`)
    - Push to main!
    - Comment in closed PR threads an update with the newly published version.

2. New release in github

    - Open your project's releases page and click 'Draft/Create a new release' (ex. https://github.com/AndcultureCode/AndcultureCode/releases/new)
    - Enter correct release version in the 'Tag version' field
    - Copy our conventional [RELEASES-TEMPLATE](./RELEASES-TEMPLATE.md) into the release description and update accordingly
    - Try your best to include all PRs and contributors

3. Double check that any build artifacts you created are in fact pushed after you published them to their respective repository (ie. npm, nuget). It is easy to forget this step.

## Dotnet Core via NuGet

*   Ensure your `.csproj` has the following details in the `<PropertyGroup>`
    ```xml
    <Company>andculture</Company>
    <Description>Some meaningful description of the package</Description>
    <Authors>andculture</Authors>
    <PackageIconUrl>https://avatars3.githubusercontent.com/u/32297579?s=460&amp;v=4</PackageIconUrl>
    <PackageId>AndcultureCode.{Language}.{ProjectName}</PackageId>
    <PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>
    <PackageProjectUrl>https://github.com/AndcultureCode/{RepositoryName}</PackageProjectUrl>
    <Version>{3 Digit Release #}</Version>
    ```
*   Update project's `.csproj` file `<Version>` using [semantic versioning](https://docs.microsoft.com/en-us/nuget/concepts/package-versioning)
*   Package your project with `dotnet pack` to create nupkg file
    *   Now you should have a `.nupkg` file in your `bin/Debug` directory
*   Publish to nuget.org using one of two methods
    *   Manual
        *   Log into nuget.org
        *   Upload previously generated `.nupkg` file
    *   Automatic (cli)
        *   Log into nuget.org
        *   Go to profile > API Keys
        *   Create new key with your name
        *   Select packages to manage
        *   Copy key and store in password locker
        *   Configure nuget locally with key
            *   `nuget setapikey <apikey> -Source https://api.nuget.org/v3/index.json`
        *   Shell to directory containing .nupkg
        *   Use dotnet cli to publish new nupkg
            *   `dotnet nuget push AndcultureCode.CSharp.{ProjectName}.{3 Digit Release #}.nugkg -s https://api.nuget.org/v3/index.json`

## Dotnet Framework via NuGet

*   If your project does not yet contain a `.nuspec` file, run the following command to generate one
    `nuget spec`   
    Note: If generating the .nuspec file for the first step, make sure the following `/Properties/AssemblyInfo.cs` attributes are set appropriately so that `.nuspec` can pull those in for token replacement:
    *   `AssemblyTitle`  
    *   `AssemblyDescription`  
    *   `AssemblyCompany`
    *   `AssemblyProduct`
    *   `AssemblyCopyright`
*   Ensure your `/Properties/AssemblyInfo.cs` has a 3 digit release number specified in the following lines of code
    ```c#
    [assembly: AssemblyVersion("{3 Digit Release #}.0")]
    [assembly: AssemblyFileVersion("{3 Digit Release #}.0")]
    ```
*   Package your project with `nuget pack` to create nupkg file
    *   `nuget pack src/AndcultureCode.CSharp.{ProjectName}/{ProjectName}.csproj`
    *   Now you should have a `.nupkg` file in the root directory of your project
*   Publish to nuget.org using one of two methods
    *   Manual
        *   Log into nuget.org
        *   Upload previously generated `.nupkg` file
    *   Automatic (cli)
        *   Log into nuget.org
        *   Go to profile > API Keys
        *   Create new key with your name
        *   Select packages to manage
        *   Copy key and store in password locker
        *   Configure nuget locally with key
            *   `nuget setapikey <apikey> -Source https://api.nuget.org/v3/index.json`
        *   Shell to directory containing .nupkg
        *   Use dotnet cli to publish new nupkg
            *   `nuget push AndcultureCode.CSharp.{ProjectName}.{3 Digit Release #}.nugkg -src https://api.nuget.org/v3/index.json`

## JavaScript via NPM

*   Login with npm in your terminal using organization credentials in password locker
    *   `npm login`
*   Change directory to the folder containing the project `package.json`
*   Update `package.json` version number using [semantic versioning](https://docs.npmjs.com/about-semantic-versioning)
*   Use npm to publish public npm registry
    *   `npm publish`
