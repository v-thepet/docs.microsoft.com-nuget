---
title: How to create a NuGet package
description: A detailed guide to the process of designing and creating a NuGet package, including key decision points like files and versioning.
author: kraigb
ms.author: kraigb
manager: douge
ms.date: 05/24/2018
ms.topic: conceptual
---

# Creating NuGet packages

[TODO: the folder works for nuget spec and nuget pack; if you just have info in the project file, you can not have a nuspec at all...but to waht extent can you customize?]

A NuGet package is simply a way to distribute libraries of code or software for a wide variety of uses. Technically speaking, a NuGet package is just a ZIP file that uses the `.nupkg` extension and whose contents match certain conventions. As described in the [Overview and workflow](Overview-and-Workflow.md) article, NuGet packages can contain a variety of different files, such as .NET assemblies, symbol files, JavaScript and CSS files for web applications, tools, and any other other arbitrary content. "SDK" packages, whose sole purpose is to bring in other packages as dependencies, don't even have any content of their own!

No matter what your package does or what code it contains, the package creation process follows the same general steps as described in this article:

1. Organize the files you want to include in the package into a conventional folder structure.

1. Run the `nuget spec` command to generate a *package manifest*, which is a `.nuspec` file, that contains package metadata.

1. Edit the manifest to contain all of the specific information for your package, such as identifier, version number, copyright information, tags, and so on.

1. Run the `nuget pack` command to create the package from the manifest and the folder structure. The `nuget pack` command automatically includes all the files in the folder structure.

Because many packages contains only a single .NET assembly that targets a single framework (like .NET Standard 2.0), Visual Studio and the .NET CLI provide streamlined support for package creation. If you're creating such a package, see the Quickstart articles for [.NET Standard with the dotnet CLI](../quickstart/create-and-publish-a-package-using-the-dotnet-cli.md), [.NET Standard with Visual Studio](../quickstart/create-and-publish-a-package-using-visual-studio.md), and [.NET Framework with Visual Studio](../quickstart/create-and-publish-a-package-using-visual-studio-net-framework.md). For packages that contain any other variation, however, you need to follow the process described in this present article.

Also, you can also use the `dotnet pack` and `msbuild /t:pack` commands to build a package based on the contents of a project file. This process is described separately in [TODO].

## Organize files into a conventional folder structure

When NuGet installs a package, it performs different actions on the package's content depending on the folders in which that content is placed in the package:

| Folder | Description | Action upon package install |
| --- | --- | --- |
| (root) | Package root | Files placed here are ignored with the exception of `readme.txt` that Visual Studio displays after installing the package. |
| lib/{TFM} | Assembly (`.dll`), documentation (`.xml`), and symbol (`.pdb`) files for the given .NET Target Framework Moniker (TFM) | Assemblies are added as references; `.xml` and `.pdb` files are copied into project folders. See [Supporting multiple target frameworks](supporting-multiple-target-frameworks.md) for creating framework target-specific sub-folders. |
| runtimes | Architecture-specific assembly (`.dll`), symbol (`.pdb`), and native resource (`.pri`) files | Assemblies are added as references; other files are copied into project folders. See [Supporting multiple target frameworks](supporting-multiple-target-frameworks.md). |
| content | Arbitrary files | Contents are copied to the project root. Think of the **content** folder as the root of the target application that ultimately consumes the package. For example, to have the package add JavaScript files to the application's */scripts* folder, place those files in the package's *content/scripts* folder. |
| build | MSBuild `.targets` and `.props` files | Automatically inserted into the project file. |
| tools | Powershell scripts and programs accessible from the Package Manager Console | The `tools` folder is added to the `PATH` environment variable for the Package Manager Console only (Specifically, *not* to the `PATH` as set for MSBuild when building the project). For projects using `packages.config`, the `tools` folder can also contain PowerShell scripts named `install.ps1` and `uninstall.ps1`, which are run after the package is installed and before the package is uninstalled, respectively.|

TODO: transforms?

> [!Note]
> NuGet 2.x supported the notion of a solution-level package that installs tools or additional commands for the Package Manager Console (the contents of the `tools` folder), but does not add references, content, or build customizations to any projects in the solution. Such packages contain no files in its direct `lib`, `content`, or `build` folders, and none of its dependencies have files in their respective `lib`, `content`, or `build` folders. In this case, NuGet tracks installed solution-level packages in a `packages.config` file in the `.nuget` folder, rather than the project's `packages.config` file.
>
> Solution-level packages are deprecated in NuGet 3.0 and later.

## Create the manifest

Once you know what files you want to package, the next step is creating a package manifest, which is an XML file with the `.nuspec` extension. Manifests are often referred to simply as a "nuspec".

This section first explains the role of the manifest and its structure, then outlines the different ways to create the file.

### The role and structure of the manifest

The manifest serves the following purposes:

1. Describes the package's contents and is itself included in the package.
1. Drives both the creation of the package and instructs NuGet on how to install the package into a project. For example, the manifest identifies other package dependencies such that NuGet can also install those dependencies when the main package is installed.
1. Contains both required and optional properties as described below. For exact details, including other properties not mentioned here, see  the [.nuspec reference](../reference/nuspec.md).

Required properties:

- The package identifier, which must be unique across the gallery that hosts the package.
- A specific version number in the form *Major.Minor.Patch[-Suffix]* where *-Suffix* identifies [pre-release versions](prerelease-packages.md)
- Author and owner information.
- A long description of the package.

Common optional properties:

- The package title as it should appears on the host (like nuget.org)
- Release notes
- Copyright information
- A short description for the [Package Manager UI in Visual Studio](../tools/package-manager-ui.md)
- A locale ID
- Home page and license URLs
- An icon URL
- Lists of dependencies and references
- Tags that assist in gallery searches

The following is a typical (but fictitious) `.nuspec` file, with comments describing the properties:

```xml
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
    <metadata>
        <!-- The identifier that must be unique within the hosting gallery -->
        <id>Contoso.Utility.UsefulStuff</id>

        <!-- The package's title that may be shown on the hosting gallery -->
        <title>A library of useful utilities.</title>

        <!-- The package version number that is used when resolving dependencies -->
        <version>1.8.3-beta</version>

        <!-- Authors contain text that appears directly on the gallery -->
        <authors>Dejana Tesic, Rajeev Dey</authors>

        <!-- 
            Owners are typically nuget.org identities that allow gallery
            users to easily find other packages by the same owners.  
        -->
        <owners>dejanatc, rjdey</owners>

         <!-- License and project URLs provide links for the gallery -->
        <licenseUrl>http://opensource.org/licenses/MS-PL</licenseUrl>
        <projectUrl>http://github.com/contoso/UsefulStuff</projectUrl>

        <!-- The icon is used in Visual Studio's package manager UI -->
        <iconUrl>http://github.com/contoso/UsefulStuff/nuget_icon.png</iconUrl>

        <!-- 
            If true, this value prompts the user to accept the license when
            installing the package. 
        -->
        <requireLicenseAcceptance>false</requireLicenseAcceptance>

        <!-- Any details about this particular release -->
        <releaseNotes>Bug fixes and performance improvements</releaseNotes>

        <!-- 
            The description can be used in package manager UI. Note that the
            nuget.org gallery uses information you add in the portal. 
        -->
        <description>Core utility functions for web applications</description>

        <!-- Copyright information -->
        <copyright>Copyright ©2016 Contoso Corporation</copyright>

        <!-- Tags appear in the gallery and can be used for tag searches -->
        <tags>web utility http json url parsing</tags>

        <!-- Dependencies are automatically installed when the package is installed -->
        <dependencies>
            <dependency id="Newtonsoft.Json" version="9.0" />
        </dependencies>
    </metadata>

    <!-- A readme.txt to display when the package is installed -->
    <files>
        <file src="readme.txt" target="" />
    </files>
</package>
```

A minimal manifest with only the required properties is as follows:

```xml
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
    <metadata>
        <id>Contoso.Utility.UsefulStuff</id>
        <version>1.8.3-beta</version>
        <authors>Dejana Tesic, Rajeev Dey</authors>
        <owners>dejanatc, rjdey</owners>
        <description>Core utility functions for web applications</description>
    </metadata>
</package>
```

For details on declaring dependencies and specifying version numbers, see [Package versioning](../reference/package-versioning.md). It is also possible to surface assets from dependencies directly in the package by using the `include` and `exclude` attributes on the `dependency` element. See [.nuspec Reference - Dependencies](../reference/nuspec.md#dependencies).

Because the manifest is included in the package created from it, you can find any number of additional examples by examining existing packages. A good source is the *global-packages* folder on your computer, the location of which is returned by the following command:

```cli
nuget locals -list global-packages
```

Go into any *package\version* folder and you should see the `.nuspec` file for that package.

### Creating the .nuspec file

Although you can author a manifest file from scratch, it's best to generate the initial file using the `nuget spec` command. You then edit the file to describe the exact content you want in the final package. Editing is necessary because the `nuget pack` command fails if you do not replace the default values that `nuget spec` generates.

The `nuget spec` command behaves as follows:

| Command | Behavior |
| --- | --- |
| `nuget spec <package_id>` | Generates a file named `<package_id>.nuspec` containing default values and `<package_id>` in the `<id>` field. For example, `nuget spec Contoso.Utility.UsefulStuff` generated `Contoso.Utility.UsefulStuff.nuspec`. If `<package_id>` is omitted, the name "Package" is used. For an example, see [nuget spec command - default manifest](../tools/cli-ref-spec.md#default-manifest). |
| `nuget spec` in a folder containing a project file<br/>(`.csproj`, `.vbproj`, etc.) | Generates a `.nuspec` file using the same name as the project file. The manifest contains *tokens* that are replaced at packaging time with values from the project, including references to any other packages that have already been installed. See [Tokens](#tokens) below for more information. |
| `nuget spec <assembly-name>.dll` | Generates a file that uses metadata in the assembly instead of default values. For example, the `<id>` property is set to the assembly name, and `<version>` is set to the assembly version. Other properties in the manifest, however, don't have matching values in the assembly and thus still contain placeholders ("Microsoft" is used as the author and owner). For an example, see [nuget spec command - manifest generated from an assembly](../tools/cli-ref-spec.md#manifest-generated-from-an-assembly). |

#### Tokens

When you create a manifest from a project file, tokens that refer to values in the project file are delimited by `$` symbols on both sides of the project property. For example, the `<id>` value in a manifest generated in this way typically appears as follows:

```xml
<id>$id$</id>
```

For an example of the file generated with tokens, see [nuget spec command - manifest using project tokens](../tools/cli-ref-spec.md#manifest-using-project-tokens).

Tokens are replaced with appropriate values from the project file at packing time. The $id$ token, for example, is replaced with the value of the `AssemblyName` in the project file. For the exact mapping of project values to `.nuspec` tokens, see the [Replacement Tokens reference](../reference/nuspec.md#replacement-tokens).

Tokens relieve you from needing to update crucial values like the version number in the `.nuspec` as you update the project. (You can always replace the tokens with literal values, if desired).

## Edit the manifest

Before running `nuget pack`, you must edit the manifest to remove all placeholder values and complete at least the required properties described earlier under [the role and structure of the manifest](#the-role-and-structure-of-the-manifest). The example manifest given in that section provides a description of the properties.

The package identifier (`<id>` element) and the version number (`<version>` element) are the two most important values in the manifest because they uniquely identify the exact code that's contained in the package.

Best practices for package identifiers:

- **Uniqueness**: The identifier must be unique across nuget.org or whatever gallery hosts the package. Before deciding on an identifier, search the applicable gallery to check if the name is already in use. To avoid conflicts, a good pattern is to use your company name as the first part of the identifier, such as `Contoso.`.
- **Namespace-like names**: Follow a pattern similar to namespaces in .NET, using dot notation instead of hyphens or underlines. Consumers also find it helpful when the package identifier matches the namespaces used in the code. Examples:

    | Naming | Best practice? |
    |`Contoso.Utility.UsefulStuff` | Yes |
    |`Contoso-Utility-UsefulStuff` | No |
    |`Contoso_Utility_UsefulStuff` | No |

- **Sample Packages**: If you produce a package of sample code that demonstrates how to use another package, attach `.Sample` as a suffix to the identifier, as in `Contoso.Utility.UsefulStuff.Sample`. (The sample package would of course have a dependency on the other package.) When creating a sample package, use the convention-based working directory method described earlier. In the `content` folder, arrange the sample code in a folder called `\Samples\<identifier>` as in `\Samples\Contoso.Utility.UsefulStuff.Sample`.

- **SDK Packages**: SDK packages contain no content of their own but only declare dependencies on other packages to install them as a group. Use a name that identifies the package as an SDK, typically with the term "SDK" somewhere in the name.

Best practices for package versions:

- When the package contains a .NET assembly, set the version of the package to match that of the assembly. Using the $version$ token to use the value from a project file is a good way to accomplish this match. However, matching versions is not strictly required because NuGet uses only package versions when resolving dependencies, not assembly versions.
- When using a non-standard version scheme, be sure to consider the NuGet versioning rules as explained in [Package versioning](../reference/package-versioning.md).
- To assign a pre-release version, see [Pre-release versions](prerelease-packages.md).

Editing the manifest is typically necessary to extend the capabilities of your package as described in the following articles:

- [Include content files](content-files.md)
- [Supporting multiple target frameworks](supporting-multiple-target-frameworks.md)
- [Transformations of source and configuration files](source-and-config-file-transformations.md)
- [Localization](creating-localized-packages.md)
- [Native Packages](native-packages.md)
- [Symbol Packages](symbol-packages.md)

<a name="creating-the-package"></a>

## Running nuget pack to generate the .nupkg file

If your manifest does not contain tokens, create a package by running `nuget pack` with your `.nuspec` file, replacing `<name>` with your specific filename:

```cli
nuget pack <name>.nuspec
```

When using a manifest with tokens, run `nuget pack` with your *project file*, which automatically loads the matching `.nuspec` file and replaces any tokens within it using values in the project file. (Token replacement does *not* happen if you use `nuget pack` with a `.nuspec` file.)

```cli
nuget pack <project-name>.csproj
```

In all cases, `nuget pack` excludes folders that start with a period, such as `.git` or `.hg`.

NuGet indicates if there are any errors in the `.nuspec` file that need correcting, such as forgetting to change placeholder values.

Once `nuget pack` succeeds, you have a `.nupkg` file that you can publish to a suitable gallery as described in [Publishing a Package](../create-packages/publish-a-package.md).

> [!Tip]
> A helpful way to examine a package after creating it is to open it in the [Package Explorer](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer) tool. This gives you a graphical view of the package contents and its manifest. You can also add the `.zip` extension to the file and explore its contents directly.

### Other packing tools

Instead of `nuget pack`, you can use `msbuild /t:pack` with MSBuild 15.1+. By default, MSBuild uses information from the project file only, but you can refer to a `.nuspec` file within the project file. For more information, see [NuGet pack and restore as MSBuild targets - pack target](../reference/msbuild-targets.md#pack-target), specifically [Packing using a .nuspec](../reference/msbuild-targets.md#packing-using-a-nuspec).

You can also use the `dotnet pack` command, which operates like MSBuild, and you can refer to a `.nuspec` file in the same way. [TODO--verify]

### Additional options

You can use various command-line switches with `nuget pack` to exclude files, override the version number in the manifest, and change the output folder, among other features. For a complete list, refer to the [pack command reference](../tools/cli-ref-pack.md).

A few common options with Visual Studio projects are:

- **Referenced projects**: If the project references other projects, you can add the referenced projects as part of the package, or as dependencies, by using the `-IncludeReferencedProjects` option:

    ```cli
    nuget pack MyProject.csproj -IncludeReferencedProjects
    ```

    This inclusion process is recursive, so if `MyProject.csproj` references projects B and C, and those projects reference D, E, and F, then files from B, C, D, E, and F are included in the package.

    If a referenced project includes a `.nuspec` file of its own, then NuGet adds that referenced project as a dependency instead.  You need to package and publish that project separately.

- **Build configuration**: By default, NuGet uses the default build configuration set in the project file, typically *Debug*. To pack files from a different build configuration, such as *Release*, use the `-properties` option with the configuration:

    ```cli
    nuget pack MyProject.csproj -properties Configuration=Release
    ```

- **Symbols**: to include symbols that allow consumers to step through your package code in the debugger, use the `-Symbols` option:

    ```cli
    nuget pack MyProject.csproj -symbols
    ```

### Testing package installation

Before publishing a package, you typically want to test the process of installing a package into a project. The tests make sure that the necessarily files all end up in their correct places in the project.

You can test installations manually in Visual Studio or on the command line using the normal [package installation steps](../consume-packages/ways-to-install-a-package.md).

For automated testing, the basic process is as follows:

1. Copy the `.nupkg` file to a local folder.
1. Add the folder to your package sources using the `nuget sources add -name <name> -source <path>` command (see [nuget sources](../tools/cli-ref-sources.md)). Note that you need only set this local source once on any given computer.
1. Install the package from that source using `nuget install <packageID> -source <name>` where `<name>` matches the name of your source as given to `nuget sources`. Specifying the source ensures that the package is installed from that source alone.
1. Examine your file system to check that files are installed correctly.

## Next Steps

Once you've created a package, which is a `.nupkg` file, you can publish it to the gallery of your choice as described on [Publishing a Package](publish-a-package.md).
