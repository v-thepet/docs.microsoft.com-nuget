---
title: NuGet CLI spec command
description: Reference for the nuget.exe spec command
author: kraigb
ms.author: kraigb
manager: douge
ms.date: 01/18/2018
ms.topic: reference
---

# spec command (NuGet CLI)

**Applies to:** package creation &bullet; **Supported versions:** all

Generates a `.nuspec` file for a new package. If run in the same folder as a project file (`.csproj`, `.vbproj`, `.fsproj`), `spec` creates a tokenized `.nuspec` file. 

## Usage

```cli
nuget spec [<packageID>] [options]
```

where `<packageID>` is an optional identifier that is used in the `<id>` properties of the file. The file is also named `<packageID>.nuspec`. Otherwise the default identifier is "Package".

## Options

| Option | Description |
| --- | --- |
| AssemblyPath | Specifies the path to the assembly to use for metadata. |
| Force | Overwrites any existing `.nuspec` file. |
| ForceEnglishOutput | *(3.5+)* Forces nuget.exe to run using an invariant, English-based culture. |
| Help | Displays help information for the command. |
| NonInteractive | Suppresses prompts for user input or confirmations. |
| Verbosity | Specifies the amount of detail displayed in the output: *normal*, *quiet*, *detailed*. |

Also see [Environment variables](cli-ref-environment-variables.md)

## Examples

### Default manifest

Running `nuget spec` in a folder that doesn't contain a project file generates a file containing default properties. If the package identifier is omitted, "Package" is used as the default.

Command:

```cli
nuget spec MyPackage
```

Result in `MyPackage.nuspec`, where CURRENT_USER is replaced with the current user's name. All other values besides the identifier are placeholders.

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>MyPackage</id>
    <version>1.0.0</version>
    <authors>CURRENT_USER</authors>
    <owners>CURRENT_USER</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package description</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2018</copyright>
    <tags>Tag1 Tag2</tags>
    <dependencies>
      <dependency id="SampleDependency" version="1.0" />
    </dependencies>
  </metadata>
</package>
```

For an example of the file generated with tokens, see [nuget spec command - manifest using project tokens](../tools/cli-ref-spec.md#manifest-using-project-tokens).

### Manifest using project tokens

When running `nuget spec` in a folder containing a project file, the generated manifest uses tokens to refer to values in the project file.

```cli
# Run in a folder containing a project file (MyProject.csproj, etc.):
nuget spec
```

Result in `MyProject.nuspec`:

The identifier, version, title, authors (owners), and description properties are taken from the project when `nuget pack` is run:

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2018</copyright>
    <tags>Tag1 Tag2</tags>
  </metadata>
</package>
```

### Manifest generated from an assembly

When running `nuget spec` with an assembly, the resulting manifest contains the identifier and version number taken from the assembly. 

```cli
nuget spec -AssemblyPath MyAssembly.dll
```

If MyAssembly.dll contains version is 1.3.1, the generated manifest appears as follows:

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>MyAssembly</id>
    <version>1.3.1</version>
    <authors>Microsoft</authors>
    <owners>Microsoft</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package description</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2018</copyright>
    <tags>Tag1 Tag2</tags>
    <dependencies>
      <dependency id="SampleDependency" version="1.0" />
    </dependencies>
  </metadata>
</package>
```