Automatic dependency injection, similar to .NET's Managed Extensibility Framework, but with (hopefully) less boilerplate.

Usage
=====

Declaring an interface
----------------------

{{{
trait MyTrait
}}}

Exporting a class
-----------------

{{{
@export
case class Exported extends MyTrait
}}}

Exporting a singleton
---------------------

{{{
@export
object ExportedSingleton extends MyTrait
}}}

Importing
---------

{{{
assert(Try(Import.one[MyTrait]).isFailure)
assert(Try(Import.optional[MyTrait]).isFailure)
assert(Import.all[MyTrait] == Set(Exported(), ExportedSingleton))
assert(Import.atLeastOne[MyTrait] == Set(Exported(), ExportedSingleton))
}}}

Verifying all imports on startup
--------------------------------

It's suggested (but not required) that imports are verified on application startup, this can be done like the following. Note that this will cause issues if combined with the testing techniques detailed below.

{{{
try {
    Import.verify()
} catch {
    case ex => println(s"Imports will fail: $ex")
}
}}}

Mocking
=======

Importer is an interface that, roughly, takes an interface and then returns a list of implementations. The system importer is global, but can be overridden thread-locally, which is primarily suited for testing.

Replacing exports by type
-------------------------

You can force a specific set of exports by trait.

{{{
trait OtherTrait

object TestExported extends MyTrait with OtherTrait

Importer.withReplacedExports(typeTag[MyTrait] -> Set(TestExported)) {
    assert(Import.one[MyTrait] == TestExported)
    assert(Import.optional[OtherTrait] == None)
}
}}}

Adding exports by type
----------------------

You can add a specific export by trait.

{{{
Importer.withExportsByType(typeTag[MyTrait] -> Set(TestExported)) {
    assert(Import.all[MyTrait] == Set(Exported(), ExportedSingleton, TestExported))
    assert(Import.optional[OtherTrait] == None)
}
}}}

Adding exports
-------------

You can also add a specific export for all it's traits (as if it had been marked with @export)

{{{
Importer.withExports(TestExported) {
    assert(Import.all[MyTrait] == Set(Exported(), ExportedSingleton, TestExported))
    assert(Import.one[OtherTrait] == TestExported)
}
}}}

Clearing exports

Gotchas
-------

Using singletons that store imported mocked objects will cause these mocks to leak. This can be resolved in two ways, either by not storing permanent references to imports in singletons (use `def` instead of `val`), or by manually forcing a specific importer (use `Import(Importer.default)` instead of `Import`, with the side effect of that mocks are ignored completely).
