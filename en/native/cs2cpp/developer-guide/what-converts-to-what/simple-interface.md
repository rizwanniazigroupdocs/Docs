---
date: "2019-10-11"
author:
  display_name: "xwiki:XWiki.farooqsheikh"
draft: "false"
toc: true
title: "Simple Interface"
linktitle: "Simple Interface"
menu:
  docs:
    parent: "What Converts to What"
    weight: "29"
lastmod: "2019-05-28"
weight: "29"
---

This example demonstrates how C# interfaces are ported to C++. They become C++ classes which are inherited from System::Object and have RTTI declared.

Additional command-line options passed to CsToCppPorter: none.

## Source Code ##

{{< gist csportertotal 2835382f1599d4367c1fb19f46dd15ae "csPortercpp_Csharp_SimpleInterface.cs">}}

## Ported Code ##

### C++ Header ###

{{< gist csportertotal 828e6770a3d27de2e78022affa71bfbf "csPortercpp_Cpp_SimpleInterface_Header.cpp">}}

### C++ Source Code ###

{{< gist csportertotal 828e6770a3d27de2e78022affa71bfbf "csPortercpp_Cpp_SimpleInterface.cpp">}}
