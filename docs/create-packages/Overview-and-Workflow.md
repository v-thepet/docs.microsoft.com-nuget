---
title: Overview and workflow of creating NuGet packages
description: An overview of the process of creating and publishing a NuGet package, with links to other specific parts of the process.
author: kraigb
ms.author: kraigb
manager: douge
ms.date: 05/04/2018
ms.topic: conceptual
---

# Package creation workflow

A NuGet package, in its simplest sense, is a delivery vehicle for files that other developers might want to incorporate into their projects. Many packages certainly contain .NET assemblies that contain useful code, but a package isn't required to contain any code at all. Some packages, such as jQuery and Bootstrap for example, contain only JavaScript and CSS files that are useful in a web application written in ASP.NET. Other packages contain only symbol (`.pdb`) files to aid debugging. And some packages contain nothing more than a list of other packages as dependencies. Such packages provide a singular means to install all of those other packages as a group, and is a convenient way to deliver an SDK that's composed of multiple independent packages.

Regardless of what the package itself contains, the general creation process is the same:

1. Decide what files you want to include in the package.
1. Organize those files into a conventional folder structure.
1. Run the `nuget spec` command to generate a *package manifest*, which is a `.nuspec` file, that contains package metadata.
1. Edit the manifest to contain all of the specific information for your package, such as identifier, version number, copyright information, tags, and so on.
1. Run a `pack` command (in the NuGet CLI, the dotnet CLI, or MSBuild) to create the package from the manifest and the folder structure. The resulting `.nupkg` file is actually just a `.zip` file that contains the package contents along with the manifest.

The core details of this process, including arbitrary content files, are described on [Creating a package](../create-packages/creating-a-package.md).

> [!Tip]
> Both the dotnet CLI and Visual Studio provide streamlined methods to create a package that contains only a single .NET assembly. For more information, see the following article:
> - [Quickstart: Create and publish a .NET Standard package using the dotnet CLI](../quickstart/create-and-publish-a-package-using-the-dotnet-cli.md)
> - [Quickstart: Create and publish a .NET Standard package using Visual Studio](../quickstart/create-and-publish-a-package-using-visual-studio.md)
> - [Quickstart: Create and publish a .NET Framework package using Visual Studio](../quickstart/create-and-publish-a-package-using-visual-studio-net-framework.md)

From there, you can consider a number of other options for your package:

- [Pre-release Packages](../create-packages/prerelease-packages.md) demonstrates how to release alpha, beta, and rc packages to those customers who are interested.
- [Package versioning](../reference/package-versioning.md) discusses how to identify the exact versions that you allow for your dependencies (other packages that you consume from your package).
- [Supporting Multiple Target Frameworks](../create-packages/supporting-multiple-target-frameworks.md) describes how to create a package with multiple variants for different .NET Frameworks.
- [Creating Localized Packages](../create-packages/creating-localized-packages.md) describes how to structure a package with multiple language resources and how to use separate localized satellite packages.
- [Source and Config File Transformations](../create-packages/source-and-config-file-transformations.md) describes how you can do both one-way token replacements in files that are added to a project, and modify `web.config` and `app.config` with settings that are also backed out when the package is uninstalled. (Supported only in projects that use the `packages.config` reference format.)
- [Symbol Packages](../create-packages/symbol-packages.md) offers guidance for supplying symbols for your library that allow consumers to step into your code while debugging.
- [Native Packages](../create-packages/native-packages.md) describes the process for creating a package for C++ consumers.
- [Signing Packages](../create-packages/sign-a-package.md) describes the process for adding a digital signature to a package.

## Publishing a package

When you're then ready to publish a package to nuget.org, follow the simple process in [Publish a package](../create-packages/publish-a-package.md).

If you want to use a private feed instead of nuget.org, see the [Hosting Packages Overview](../hosting-packages/overview.md).

## Making a commitment to other developers

When you create a package for use by other developers, it's important to understand that when they use your package, they are taking a dependency on your work. As such, creating and publishing a package also implies a commitment to fixing bugs and making other updates. If possible, consider making the package available as open source so others can help to maintain it.
