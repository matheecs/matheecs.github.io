---
layout: post
title: "[转]C++ 编译加速"
categories: study
author: "Jixiang Zhang"
---

Original CITE <https://devtalk.blender.org/t/speed-up-c-compilation/30508>

Shorter build times improve developer productivity. Therefore, it is benefitial to reduce them if possible. This document explains why compilation can be slow and discusses different approaches to speed it up.

This is mostly meant to be a resource for other developers that are investigating compile time issues. However, it would be nice to hear other peoples thoughts for how we can or should improve build times as well.

## Build Process

When analysing compile times, three parts of the build process are particularly interesting. Those are briefly described below.

### Source Code Parsing

This is commonly referred to as the compiler front-end. First, it run the preprocessor per translation unit. This inlines all included header files, usually resulting in a fairly large file (often >100,000 lines).

The preprocessor itself is very fast, however parsing that many lines of code is slow. Parsing C++ code is slower than parsing C code because C++ is a more complex language. At the end of parsing, the compiler has an internal representation (e.g. an AST) of the current translation unit and all included headers.

### Code Generation

Now the compiler checks which functions have to be generated, because they may be called by other translation units. Then it generates machine code for those functions, which also involves processing all the other functions in the same translation unit that are called indirectly. This is often referred to as the compiler back-end.

The compile time here mainly depends on how much machine code is generated (and not on the size of the preprocessed file). In C++ this often results in many (templated) inline functions to be generated. In C code, many of those functions would not be inline which makes C code generation generally faster given the same amount of code. In C++ they often have to be inline because of the templating mechanism.

### Linking

The linker reads the symbol table of every translation unit, i.e. the list of functions and variables that a translation unit uses and provides. Then it copies every used function into the final executable ones and updates function pointers.

The linking time mainly depends on how fast the linker can read and deduplicate all the symbol tables and how fast it can copy and link all the functions in the final executable.

The more translation units and the larger their corresponding machine code size and symbol table is, the slower the linking process.

## Redundant Work

The main reason for why building big C++ projects is slow is that the compiler and linker do a huge amount of redundant work. The same headers are parsed over and over again, and the same (templated) inlined functions are generated many times. This leads to a lot of duplicate functions in the generated object files and the linker’s job is to reduce the redundancy again. The redundant work is often much more than the work that is unique per translation unit.

## Build Time Reduction Approaches

There are different techniques to reduce build times. Generally they work by either reducing redundant work, reducing unique work or by doing the same work faster. Reducing unique work usually often involves a run-time performance trade-off.

### Faster Linker

Linking is the last step of the build process and has to be done even of only a single source file changed. Usually it has to relink everything from scratch after every change, unless something like incremental linking is used (which I don’t have experience with in Blender).

Many linkers are mostly single threaded and are therefore a huge serial bottleneck at the end of the build process.

If you can, use the mold 130 linker. I’ve use it for a long time now and it’s significantly faster then everything else I’ve tried. People have reported issues with it doing wrong things, but I’ve never experienced those. Unfortunately, it’s not available on windows yet. The new linker in Xcode 15 on macos seems to be significantly faster than in previous versions as well.

### Ccache

Build systems usually decide to rebuild files based on their last modification time on the file system. This becomes a problem when one often updates the last modification time without actually changing the source code. Ccache 30 wraps another compiler like gcc and adds a caching layer on top of it. It first checks if the source code to be compiled actually changed. If it did, the underlying compiler is invoked. Otherwise, it just outputs the result of a previous compiler invocation.

This caching layer helps in a few situations. Obviously, it avoids recompilation when just saving over an existing file without any changes. It also helps when just changing comments, because Ccache can check if the preprocessor output has changed and the preprocessor removes comments. Most importantly, it helps when switching back and forth between branches. Recompilation after switching to a branch that has been compiled before uses the cache and is thus much faster.

Note that ccache only caches the output of the code generation step, but not the linker result. Therefore, the linker still has to run again even if source files haven’t changed.

When benchmarking compilation time, Ccache can get in the way because it makes the results unreliable. Ccache can be temporarily disabled by setting the CCACHE_DISABLE environment variable to 1 (export CCACHE_DISABLE=1).

### Multiple Git Checkouts

Having multiple git checkouts can reduce compilation times by avoiding often branch switching which leads to many code changes. One could simply copy the source code folder or use git worktrees. Each checkout should have their own build folder.

Personally, I have a separate checkout that I don’t use for compilation but only to look at other branches while working on something or to checkout very old commits, e.g. when figuring out when a particular function has been added.

When working with multiple checkouts, it’s easy to get yourself confused with which checkout you are currently working on. I solved this locally, by using vscode’s ability to configure a different theme per folder. Essentially, when I’m not in my main checkout, vscode looks different. I never accidentally changed the wrong checkout since I set this up.

### Multiple Build Folders

Having multiple build folders for different configurations reduces the compilation overhead when switching between them. For example, one should have separate build folders for release and debug builds. Partially this is setup automatically when using our make utilities. Visual Studio also creates separate build folders automatically afaik.

It can be benefitial to have even more independent build folders though. Like mentioned before, it can help to have separate build folders for each git checkout. Personally, I currently have build folders for the following configurations: release, debug, reldebug, asan, compile commands (only used for better autocompletion in vscode), clang release, (used to check for other warnings and performance differences), clang release optimized (enables extra compiler optimizations to see if they have an impact), release reference (used when comparing performance of two branches).

### Forward Declarations of Types

Forward declarations allow using types from one header without actually including the header. For example, putting struct bNodeTree; in a header makes the bNodeTree type available without having to include DNA_node_types.h. The hope is that at least some translation units that use the forward declaration don’t include DNA_node_types.h anyway. This leads to less code being in the preprocessed translation units which reduces the parsing overhead.

Forward declaring types that end up being included in most translation units does not really provide any benefits for the compile times. E.g. forward declaring common containers like Vector rarely makes sense.

Headers that only work with a forward declared type are restricted with what they can do with the type. They can’t access any of its members. That includes indirect accesses by e.g. taking or returning the type by value. Generally, only pointers and references of the type can be used. Smart pointers are supported as well as long as they don’t have to access methods of the type (like the destructor). If only a single function in a header needs access to the members, the include ends up being necessary anyway.

Forward declaring some types is harder than others. C structs are generally easy to forward declare. Nested C++ structs are impossible to forward declare. Namespaces and templates also make forward declarations harder. For example, one can’t really forward declare std::string reliably outside of the standard library. That’s because it’s actually a typedef of another type. Some standard types have forward declarations in `#include <iosfwd>`, but not all. The existence of that header also indicates that it is non trivial to forward declare these types. Also see this 22.

In some headers, we currently access data members of structs even though they are forward declared. That works because the data members are used in preprocessor macros. For example IDP_Int. This is a problem when we want to replace macros with inline functions, which will require the include.

Seeing benefits from forward declarations is often difficult in C++ code, because often code changes only have a negilible effect on compile times, making it hard to measure the overall effectiveness. This is also because the translation units often end up being so large that a single header of our own does not make a big difference. Another problem is that it is very hard to consistently use forward declarations especially in C++, because many things defined in headers need the full type. It’s difficult to use forward declarations everywhere, but very easy to introduce an include later on when it becomes necessary, rendering the previous work useless.

### Smaller Headers

The idea is to split up large headers into multiple smaller headers. The hope is that some translation units that previously included the large header, will now only include a subset of the smaller headers. This can reduce parsing time.

This can be especially benefitial when part of a header requires another header that is very large. For example, BKE_volume_openvdb.hh includes openvdb which is not used by all code that deals with volume data blocks.

Smaller headers can also help with making code easier to understand. Many large headers have sections already anyway. Splitting sections into separate headers might be a nice cleanup. This only improves compile times if the individual headers are not grouped into one bigger header again, like in ED_asset.h.

### Type Erasure

Templates generally have to be defined in headers so that each translation unit can instantiate the required versions of them. Removing template parameters from a function allows it to be defined outside of a header, reducing parsing and code generation overhead.

Obviously, many functions are templates for good reasons, but in some cases type erasure can be used without sacrificing performance. For example, a function that currently takes a callable as a template could also take a FunctionRef instead. Functions taking a `Span<T>` could take a GSpan.

Sometimes the combination of a normal and type erased code path can make sense too. E.g. parallel_for has an inlined fast path, but a type erased more complex path that is not in the header. Under some circumstances, type erasure can also reduce final binary size significantly 47.

### Explicit Instantiation

Sometimes, it’s not possible or desired to remove template parameters from a function. However, if the function does not really benefit from inlining because it is too large, there is no point in compiling it in every translation unit that uses it. Compiling it only once reduces code generation time.

Explicit instantiation can be used to make sure that a templated function is only instantiated once for a specific type. See e.g. the normalized_to_eul2 function.

This approach can also be used to move the definition of a template out of a header in which case this would also reduce parsing overhead. Then the templated function can only be used with the types that have been instantiated explicitly.

While explicit instantiation can definitly help, it’s not entirely trivial to find good candidates that have a measurable impact. Maybe some better tooling could help here.

### Reuse Common Functions

When a header already includes the functionality that one would implement in a source file, it’s better to just use the shared functionality. This may seem obvious, but it’s mentioned here because it’s not necessarily simple and requires familiarizing yourself with the available utilities. For example, many geometry algorithms have a gather 50 step.

Having more common function that are not recompiled for every use reduces the code generation redundancy. Having too many such utilities that are only rarely used can also increase the parsing overhead though.

### Use Less Forced Inlining

The compiler is fairly good at deciding which functions should be inlined. It uses various heuristics to achieve a good balance of run-time performance and binary size. Generally speaking, the compiler considers all functions that are defined in a translation unit for inlining. Using the inline keyword is mostly just information for the linker to tell it that there may be multiple definitions of a function in different translation units and that they should be deduplicated. That said, depending on the compiler, the inline keyword may change the heuristics to make inlining more likely.

Compilers also support forced inlining e.g. using __forceinline. Now the compiler generally just skips the heuristics and inlines everything anyway. That’s generally the desired behavior, but can have bad consequences when used too liberally. It can result in huge functions that are slow to compile. Compiling functions often has slower-than-linear time complexity. Also see this 16.

Better just use normal inline functions and only use force inlining when you notice that the compiler is not inlining something that would lead to better performance when inlined.

### Precompiled Headers

Most of the time spend parsing code comes from parsing the most commonly used header files. Precompiled headers allow the compiler to do some preprocessing on a set of header files so that they don’t have to parsed for every translation unit. Using precompiled headers can mostly be automated with cmake.

While precompiled headers can reduce a lot of redundant parsing time, they don’t reduce the redundancy in the code generation step.

When using precompiled headers, one can accidentally create source files that can’t be compiled on their own anymore. That’s because when a precompiled header is used in a module, all translation units include it. Fixing related issues usually just means adding more include statements. These issues are easy to find automatically by simply disabling precompiled headers temporarily.

When there are only a few translation units that use a precompiled header, using them can also hurt performance. That’s because precompiling the header is single threaded and compilation of the translation units only starts when the header compilation finished. This effect is most noticable when using unity builds with a relatively large unit size.

### Unity Builds

The redundancy during the parsing and code generation step is bad because it takes a long time relative to the non-redundant work that is unique to every translation unit. Unity builds concatenate multiple source files to a single translation unit. Now, all the headers have to be parsed once and code for inline functions has to be generated once per e.g. 10 source files. So the time spend doing redundant work relative to unique work is greatly reduced, by a factor of 10 in this case.

Unity builds can be used together with precompiled headers. This can help in modules with a very larger number of source files to get rid of the remaining redundancy when parsing headers. Overall, precompiled headers become less effective with unity builds though, because parsing overhead is reduced so much already.

Similar to precompiled headers, using unity builds can result in accidentally having files that can’t be compiled on their own anymore. This is also typically fixed by adding missing includes.

Unity builds generally require some work to work and to be safe. The fundamental problem is that symbols that were local to a single source file before, are now also available in other source files in the same unit. This can lead to name collisions which break compilation. So one has to be careful with making all source files compatible with each other. Whether one prepared files successfully can be tested by putting all source files into a single unit.

Just hoping that there are no name collisions is generally not enough, at least it doesn’t give piece of mind when any local variable can suddenly clash with local variables in another file. A solution that gives more piece of mind is to put each source file into its own namespace. Only symbols that are explicitly exposed are not defined in that namespace. We already use this approach in most node implementations. E.g. see the node_geo_mesh_to_curve_cc namespace.

This approach of file specific namespace works particularly well for nodes, because generally there is only a single function that is exposed (e.g. register_node_type_geo_mesh_to_curve). Most other source files expose multiple functions, and often the exposed functions are interleaved with local functions. In such cases, it is less convinient because the file specific namespace has to be opened and closed many times as you can see in this 4 test.

Potential solutions that make this approach more convenient could be to automate inserting the namespace scopes. Alternatively, one could organize source files so that local functions are all grouped together so that only a single namespace scope is necessary. Yet another approach could be to split up source files. For example, similar to how we have one node per file, we could also have one operator per file. Without unity builds or precompiled headers, having many small source files leads to a lot of parsing overhead, but with those, it’s less of an issue.

When using unity builds, one has to decide how many files should be concatenated per unit. This is a trade-off. Small units can allow more threads to compile the units at the same time at the cost of more parsing and code generation redundancy. Large units reduce the redundant work but less threads can be used overall. Large units are benefitial when the system has only a few cores or when the total number of files to be compiled is very large so that all cores will be busy anyway. Actually creating the units can be automated with cmake.

Often, one only works in a single source file. When changing only that one file, using unity builds lead to longer recompilation time, because all other files in the same unity are recompiled as well. Often that is not an issue, but it can be when one of the other files happens to take very long, e.g. because it uses openvdb. This can be solved in using SKIP_UNITY_BUILD_INCLUSION in cmake. It allows either excluding the files that are known to be slow to compile from units, or it can be added temporarily for the file that is currently being edited.

Using SKIP_UNITY_BUILD_INCLUSION it should also be possible to start introducing unity builds in a module gradually file by file. It also makes it possible to benefit from unity builds in modules that have some files that are very hard to prepare for unity builds.

Sometimes it’s harder for IDEs to provide good autocompletion when unity builds are used. For IDEs that use the compile commands generated by cmake (CMAKE_EXPORT_COMPILE_COMMANDS), it can be benefitial to have a separate build folder just to generate the compile commands. That build folder can have unity builds and precompiled headers disabled.

Using unity builds can also make the job of the linker easier. That’s because the linker has to parse fewer symbol tables. Within each units, are symbols are already made unique as part of the compilation process. This also reduces the total size of generated machine code, which reduces the size of the build folder.

### Distributed Builds

When one has more compute resources available outside of the work PC, one can consider to use distributed builds. Those don’t reduce redundancy (they might actually increase it), but can still improve compile times by making use of more parallelism. I don’t have much experience with this myself, but tools like distcc 13 shouldn’t be too hard to get working with Blender.

Distributed builds generally only help when building lots of files in parallel, so they are less effective when working on individual source files and only few files need to be recompiled after every change.

### C++20 Modules

The compilation speedup that can be expected from C++20 modules is probably similar to that we could get from precompiled headers. So the parse overhead can be reduced, but the code generation redundancy likely remains.

Modules have the benefit over precompiled headers that source files stay independent and are not all forced to include the same precompiled header. Thus, the problem of ending up with source files that can’t be compiled on their own, does not not exist.

The downside is of course that we are not using C++20 yet. Even if we would use it, it’s unlikely that our build chains fully support them yet. And even if they would, we’d likely still have to make fairly significant changes to our code base to make use of them.

That said, switching to C++20 can also make compilation slower overall 118. It’s unclear whether using modules can offset that slowdown or whether that slowdown goes away if C++20 implementations become more mature.

## Tools

Just like for profiling the run-time performance of a program, there are also tools that measure the build time. Those can be used to make educated guesses for which changes might impact build times the most. The tools help most to identify headers that one should avoid to include or functions whose code generation takes a significant portion of the overall build.

ClangBuildAnalyzer 108
Build Insights in Visual Studio 34, SDK 2, SDK Examples 6
include-what-you-use 106
Feel free to suggest more tools that I should add here.

## Summary

Always use the fastest linker that works for you. This is likely the most significant change you can do to improve your daily work in Blender. Forward declarations can be useful sometimes, but require significant discipline to get meaningful compile time improvements. It’s much harder in C++ than it is in C. Splitting up headers can be good for improved code quality, but likely don’t have a significant enough compile time to justify spending too much time on them.

Precompiled headers make sense in modules with many similar files. Their effectiveness is greatly reduced when unity builds are used as well. Overall, unity builds are the most effective tool to reduce redundant work and therefore improve compile times. Some code changes are necessary but their effectiveness is easily measurable.

Distributed builds are a nice thing to try for people who have the resources. C++20 modules could help like precompiled headers but are out of reach for the foreseeable future.
