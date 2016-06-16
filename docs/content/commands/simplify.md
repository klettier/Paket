Simplify will also affect paket.references files, unless [strict](dependencies-file.html#Strict-references) mode is used.

<blockquote>Important: `paket simplify` is a heuristic approach to dependency simplification. It often works very well, but there are rare cases where simplify can result in changes of the package resolution.</blockquote>

## Sample

When you install `Castle.Windsor` package in NuGet to a project, it will generate a following `packages.config` file in the project location:

    [lang=xml]
    <?xml version="1.0" encoding="utf-8"?>
    <packages>
      <package id="Castle.Core" version="3.3.1" targetFramework="net451" />
      <package id="Castle.Windsor" version="3.3.0" targetFramework="net451" />
    </packages>

After converting to Paket with [`paket convert-from-nuget command`](paket-convert-from-nuget.html), you should get a following paket.dependencies file:

    [lang=paket]
    source https://nuget.org/api/v2

    nuget Castle.Core 3.3.1
    nuget Castle.Windsor 3.3.0

and the NuGet `packages.config` should be converted to following paket.references file:

    Castle.Core
    Castle.Windsor

As you have already probably guessed, the `Castle.Windsor` package happens to have a dependency on the `Castle.Core` package.
Paket by default (without [strict](dependencies-file.html#Strict-references) mode) adds references to all required dependencies of a package that you define for a specific project in paket.references file.
In other words, you still get the same result if you remove `Castle.Core` from your paket.references file.
And this is exactly what happens after executing `paket simplify` command:

    [lang=paket]
    source https://nuget.org/api/v2

    nuget Castle.Windsor 3.3.0

will be the content of your paket.dependencies file, and:

    Castle.Windsor

will be the content of your paket.references file.

Unless you are relying heavily on components from `Castle.Core`, you would not care about controlling the required version of `Castle.Core` package. Paket will do the job.

The simplify command will help you maintain your direct dependencies.

## Interactive mode

Sometimes, you may still want to have control over some of the transitive dependencies. In this case you can use the `--interactive` flag,
which will ask you to confirm before deleting a dependency from a file.


## Notes and Warnings
It is possible through the use of simplify to make unintended changes to your paket.lock file if a transitive dependency is removed and a subsequent install is run.  This can occur especially if the version specification that previously existed for the transitive dependency is different than the version specification that is automatically generated by paket based on the primary dependencies of your project.  An example may help illustrate the conditions where this could occur.

Imagine a paket.dependencies file with the following contents:

    [lang=paket]
    source https://nuget.org/api/v2

    nuget Foo 1.0.0
    nuget Bar 1.0.0

Let us imagine that package Bar has a dependency on package Foo for any version from 1.0 through 2.0. The paket.lock file that results for this combination is:

    [lang=paket]
    NUGET
      remote: https://nuget.org/api/v2
        Foo (1.0.0)
        Bar (1.0.0)
          Foo (<= 2.0.0)

In this situation, if simplify is run, the Foo dependency will be removed, resulting in a paket.dependencies of:

    [lang=paket]
    source https://nuget.org/api/v2

    nuget Bar 1.0.0

like we expect.  But now, if an install is run, the Foo dependency will be free to update to the maximum allowed by the Bar dependency, which could have unforseen consequences if the author of Foo has introduced a breaking change.