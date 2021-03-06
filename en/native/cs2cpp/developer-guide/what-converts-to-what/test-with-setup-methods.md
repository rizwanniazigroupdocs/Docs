---
date: "2019-10-11"
author:
  display_name: "xwiki:XWiki.farooqsheikh"
draft: "false"
toc: true
title: "Test with Setup Methods"
linktitle: "Test with Setup Methods"
menu:
  docs:
    parent: "What Converts to What"
    weight: "38"
lastmod: "2019-05-28"
weight: "38"
---

This example demonstrates how NUnit test fixture with SetUp method is ported to C++. Googletest C++ library is used to translate NUnit tests to C++. SetUp, TearDown, TestFixtureSetUp, and TestFixtureTearDown methods are supported and translated to corresponding methods of googletest.

Additional command-line options passed to CsToCppPorter: none.

## Source Code ##

{{< gist csportertotal 2835382f1599d4367c1fb19f46dd15ae "csPortercpp_Csharp_TestSetupMethods.cs">}}

## Ported Code ##

### C++ Header ###

{{< gist csportertotal 828e6770a3d27de2e78022affa71bfbf "csPortercpp_Cpp_TestSetupMethods_Header.cpp">}}

### C++ Source Code ###

{{< gist csportertotal 828e6770a3d27de2e78022affa71bfbf "csPortercpp_Cpp_TestSetupMethods.cpp">}}
