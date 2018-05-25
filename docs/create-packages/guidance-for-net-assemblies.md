---
title: Guidance for structuring .NET assemblies for NuGet packages
description: Best practices for creating assemblies that are distributed through NuGet packages, including COM interop assemblies.
author: kraigb
ms.author: kraigb
manager: douge
ms.date: 05/24/2018
ms.topic: conceptual
---

# Guidance for .NET assemblies

- When possible, create assemblies for .NET Standard to provide compatibility with the widest array of project types. If you must specifically target different versions of the .NET Framework, different PCL profiles, and so forth, then you must build separate assemblies for each target. Each assembly must then be placed within the package's `lib` folder within a subfolder for the appropriate Target Framework Moniker (TFM). See [Supporting multiple target frameworks](supporting-multiple-target-frameworks.md).

- In general, it's best to have one assembly per NuGet package, provided that each assembly is independently useful. For example, if you have a `Utilities.dll` that depends on `Parser.dll`, and `Parser.dll` is useful on its own, then create one package for each. Doing so allows developers to use `Parser.dll` independently of `Utilities.dll`. The package for `Utilities.dll` will, of course, declare the parser package as a dependency.

- If your library is composed of multiple assemblies that aren't independently useful, then it's fine to combine them into one package. Using the previous example, if `Parser.dll` contains code that's used only by `Utilities.dll`, then it's fine to keep `Parser.dll` in the same package.

- Similarly, if `Utilities.dll` depends on `Utilities.resources.dll`, where again the latter is not useful on its own, then put both in the same package.

Resources are, in fact, a special case. When a package is installed into a project, NuGet automatically adds assembly references to the package's DLLs, *excluding* those that are named `.resources.dll` because they are assumed to be localized satellite assemblies (see [Creating localized packages](creating-localized-packages.md)). For this reason, avoid using `.resources.dll` for files that otherwise contain essential package code.

## Authoring packages with COM interop assemblies

Packages that contain COM interop assemblies must include an appropriate [targets file](creating-a-package.md#including-msbuild-props-and-targets-in-a-package) so that the correct `EmbedInteropTypes` metadata is added to projects using the PackageReference format. By default, the `EmbedInteropTypes` metadata is always false for all assemblies when PackageReference is used, so the targets file adds this metadata explicitly. To avoid conflicts, the target name should be unique; ideally, use a combination of your package name and the assembly being embedded, replacing the `{InteropAssemblyName}` in the example below with that value. (Also see [NuGet.Samples.Interop](https://github.com/NuGet/Samples/tree/master/NuGet.Samples.Interop) for an example.)

```xml
<Target Name="Embedding**AssemblyName**From**PackageId**" AfterTargets="ResolveReferences" BeforeTargets="FindReferenceAssembliesForReferences">
  <ItemGroup>
    <ReferencePath Condition=" '%(FileName)' == '{InteropAssemblyName}' AND '%(ReferencePath.NuGetPackageId)' == '$(MSBuildThisFileName)' ">
      <EmbedInteropTypes>true</EmbedInteropTypes>
    </ReferencePath>
  </ItemGroup>
</Target>
```

When such a package is added to a project that's using the `packages.config` management format, NuGet and Visual Studio check for COM interop assemblies and set the `EmbedInteropTypes` to true in the project file. In this case the targets are overridden.

Additionally, by default the [build assets do not flow transitively](../consume-packages/package-references-in-project-files.md#controlling-dependency-assets). Packages authored as described here work differently when they are pulled as a transitive dependency from a project to project reference. The package consumer can allow them to flow by modifying the PrivateAssets default value to not include build.
