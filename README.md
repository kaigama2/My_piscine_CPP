# My_piscine_CPP
Try yourself out😅😏💰
************************************************************************************************
title	ms.date	helpviewer_keywords	description
Overview of modules in C++
03/30/2022
modules [C++]
modules [C++], overview
Modules in C++20 provide a modern alternative to header files.
Overview of modules in C++
C++20 introduces modules, a modern solution that turns C++ libraries and programs into components. A module is a set of source code files that are compiled independently of the translation units that import them. Modules eliminate or reduce many of the problems associated with the use of header files. They often reduce compilation times. Macros, preprocessor directives, and non-exported names declared in a module aren't visible outside the module. They have no effect on the compilation of the translation unit that imports the module. You can import modules in any order without concern for macro redefinitions. Declarations in the importing translation unit don't participate in overload resolution or name lookup in the imported module. After a module is compiled once, the results are stored in a binary file that describes all the exported types, functions, and templates. The compiler can process that file much faster than a header file. And, the compiler can reuse it every place where the module is imported in a project.

You can use modules side by side with header files. A C++ source file can import modules and also #include header files. In some cases, you can import a header file as a module rather than include it textually by using #include in the preprocessor. We recommend you use modules in new projects rather than header files as much as possible. For larger existing projects under active development, experiment with converting legacy headers to modules. Base your adoption on whether you get a meaningful reduction in compilation times.

To contrast modules with other ways to import the standard library, see Compare header units, modules, and precompiled headers.

Enable modules in the Microsoft C++ compiler
As of Visual Studio 2022 version 17.1, C++20 standard modules are fully implemented in the Microsoft C++ compiler.

Before it was specified by the C++20 standard, Microsoft had experimental support for modules in the Microsoft C++ compiler. The compiler also supported importing the Standard Library as modules, described below.

Starting with Visual Studio 2022 version 17.5, importing the Standard Library as a module is both standardized and fully implemented in the Microsoft C++ compiler. This section describes the older, experimental method, which is still supported. For information about the new standardized way to import the Standard Library using modules, see Import the C++ standard library using modules.

You can use the modules feature to create single-partition modules and to import the Standard Library modules provided by Microsoft. To enable support for Standard Library modules, compile with /experimental:module and /std:c++latest. In a Visual Studio project, right-click the project node in Solution Explorer and choose Properties. Set the Configuration drop-down to All Configurations, then choose Configuration Properties > C/C++ > Language > Enable C++ Modules (experimental).

A module and the code that consumes it must be compiled with the same compiler options.

Consume C++ Standard Library as modules (experimental)
This section describes the experimental implementation, which is still supported. The new standardized way of consuming the C++ Standard Library as modules is described in Import the C++ standard library using modules.

By importing the C++ Standard Library as modules rather than including it through header files, you can potentially speed up compilation times depending on the size of your project. The experimental library is split into the following named modules:

std.regex provides the content of header <regex>
std.filesystem provides the content of header <filesystem>
std.memory provides the content of header <memory>
std.threading provides the contents of headers <atomic>, <condition_variable>, <future>, <mutex>, <shared_mutex>, and <thread>
std.core provides everything else in the C++ Standard Library
To consume these modules, add an import declaration to the top of the source code file. For example:

import std.core;
import std.regex;
To consume the Microsoft Standard Library modules, compile your program with /EHsc and /MD options.

Basic example
The following example shows a simple module definition in a source file called Example.ixx. The .ixx extension is required for module interface files in Visual Studio. In this example, the interface file contains both the function definition and the declaration. However, you can also place the definitions in one or more separate module implementation files, as shown in a later example. The export module Example; statement indicates that this file is the primary interface for a module called Example. The export modifier on f() indicates that this function is visible when Example is imported by another program or module. The module references a namespace Example_NS.

// Example.ixx
export module Example;

#define ANSWER 42

namespace Example_NS
{
   int f_internal() {
        return ANSWER;
      }

   export int f() {
      return f_internal();
   }
}
The file MyProgram.cpp uses the import declaration to access the name that is exported by Example. The name Example_NS is visible here, but not all of its members. Also, the macro ANSWER isn't visible.

// MyProgram.cpp
import Example;
import std.core;

using namespace std;

int main()
{
   cout << "The result of f() is " << Example_NS::f() << endl; // 42
   // int i = Example_NS::f_internal(); // C2039
   // int j = ANSWER; //C2065
}
The import declaration can appear only at global scope.

Module grammar
module-name:
 module-name-qualifier-seqopt identifier

module-name-qualifier-seq:
 identifier .
 module-name-qualifier-seq identifier .

module-partition:
 : module-name

module-declaration:
 exportopt module module-name module-partitionopt attribute-specifier-seqopt ;

module-import-declaration:
 exportopt import module-name attribute-specifier-seqopt ;
 exportopt import module-partition attribute-specifier-seqopt ;
 exportopt import header-name attribute-specifier-seqopt ;

Implementing modules
A module interface exports the module name and all the namespaces, types, functions, and so on that make up the public interface of the module. A module implementation defines the things exported by the module. In its simplest form, a module can consist of a single file that combines the module interface and implementation. You can also put the implementations in one or more separate module implementation files, similar to how .h and .cpp files are used.

For larger modules, you can split parts of the module into submodules called partitions. Each partition consists of a module interface file that exports a module partition name. A partition may also have one or more partition implementation files. The module as a whole has one primary module interface, the public interface of the module that may also import and export the partition interfaces.

A module consists of one or more module units. A module unit is a translation unit (a source file) that contains a module declaration. There are several types of module units:

A module interface unit is a module unit that exports a module name or module partition name. A module interface unit has export module in its module declaration.

A module implementation unit is a module unit that doesn't export a module name or module partition name. As the name implies, it's used to implement a module.

A primary module interface unit is a module interface unit that exports the module name. There must be one and only one primary module interface unit in a module.

A module partition interface unit is a module interface unit that exports a module partition name.

A module partition implementation unit is a module implementation unit that has a module partition name in its module declaration, but no export keyword.

The export keyword is used in interface files only. An implementation file can import another module, but can't export any names. Implementation files can have any extension.

Modules, namespaces, and argument-dependent lookup
The rules for namespaces in modules are the same as in any other code. If a declaration within a namespace is exported, the enclosing namespace (excluding non-exported members) is also implicitly exported. If a namespace is explicitly exported, all declarations within that namespace definition are exported.

When it does argument-dependent lookup for overload resolutions in the importing translation unit, the compiler considers functions declared in the same translation unit (including module interfaces) as where the type of the function's arguments are defined.

Module partitions
A module partition is similar to a module, except it shares ownership of all declarations in the entire module. All names exported by partition interface files are imported and re-exported by the primary interface file. A partition's name must begin with the module name followed by a colon. Declarations in any of the partitions are visible within the entire module. No special precautions are needed to avoid one-definition-rule (ODR) errors. You can declare a name (function, class, and so on) in one partition and define it in another. A partition implementation file begins like this:

module Example:part1;
The partition interface file begins like this:

export module Example:part1;
To access declarations in another partition, a partition must import it, but it can only use the partition name, not the module name:

module Example:part2;
import :part1;
The primary interface unit must import and re-export all of the module's interface partition files like this:

export import :part1;
export import :part2;
...
The primary interface unit can import partition implementation files, but can't export them. Those files aren't allowed to export any names. This restriction enables a module to keep implementation details internal to the module.

Modules and header files
You can include header files in a module source file by putting the #include directive before the module declaration. These files are considered to be in the global module fragment. A module can only see the names in the global module fragment that are in headers it explicitly includes. The global module fragment only contains symbols that are used.

// MyModuleA.cpp

#include "customlib.h"
#include "anotherlib.h"

import std.core;
import MyModuleB;

//... rest of file
You can use a traditional header file to control which modules are imported:

// MyProgram.h
import std.core;
#ifdef DEBUG_LOGGING
import std.filesystem;
#endif
Imported header files
Some headers are sufficiently self-contained that they can be brought in using the import keyword. The main difference between an imported header and an imported module is that any preprocessor definitions in the header are visible in the importing program immediately after the import statement.

import <vector>;
import "myheader.h";
See also
module, import, export
Named modules tutorial
Compare header units, modules, and precompiled headers
