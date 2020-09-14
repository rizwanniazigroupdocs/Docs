---
date: "2019-10-11"
author:
  display_name: "xwiki:XWiki.farooqsheikh"
draft: "false"
toc: true
title: "How to Convert Complex C# NUnit Test to C++"
linktitle: "How to Convert Complex C# NUnit Test to C++"
menu:
  docs:
    parent: "Porting Complex C# Projects"
    weight: "3"
lastmod: "2019-06-27"
weight: "3"
---

&nbsp;
{{<note>}}
Note that this example is built upon several assumptions, namely:
<ul>
<li>
Porter is installed to //C:\//CodePorting.Native_Cs2Cpp//_19.4// directory
</li>
<li>
All C# projects are located in //C:\ComplexNUnitTest// directory
</li>
<li>
The output directory for all projects is //C:\output//
</li>
</ul>
{{</note>}}

## Porting Complex NUnit Test ##

This example demonstrates how to port five C# projects one being an NUnit test project and another four – interdependent library projects on which the NUnit test project depends. We’ll use pre-existing projects from **ComplexNUnitTest** example located [here](https://github.com/codeporting-native/codeporting-native-cs2cpp||shape#"rect").

This example consists of five C# projets – BaseLibrary, CommonLibrary, LibraryA, LibraryB and ComplexNUnitTest. We’ll port projects one by one starting from the least dependent one.

### Porting BaseLibrary ###

BaseLibrary is a library project consisting of a single .cs source file IBaseInterface.cs and a project file BaseLibrary.csproj. This project does not have any special dependencies on other projects or 3rd party assemblies. Also BaseLibrary project directory contains pre-created configuration file BaseLibrary.porter.config. Let us have a closer look at it.

{{< highlight xml >}}
BaseLibrary.porter.config is quite simple. It begins with an XML declaration, which specifies that the file contains an XML document.

Then goes the XML root element <porter> which is mandatory for Porter configuration XML document

    <porter>

Next, the default Porter configuration file is imported using <import> element. The default configuration assigns default values to all configuration options.

    <import config="porter.config"/>

And the XML document is finished with closing tag of the root element <porter>:

    </porter>
{{< /highlight >}}

This example assumes that C# BaseLibrary library project should be ported into C++ static library project, which is a default setting.

With C# project at hand and configuration file ready, we can start porting the project.

In order to covert BaseLibrary project we run CMD and navigate to the directory with porter binary:

```
>cd C:\CodePorting.Native_Cs2Cpp_19.4\bin\porter
```

And run Porter:

```
>CsToCppPorter.exe -c C:\ComplexNUnitTest\BaseLibrary\BaseLibrary.porter.config C:\ComplexNUnitTest\BaseLibrary\BaseLibrary.csproj C:\output
```

Porter will print some logs of the porting process to the console window and when it finishes porting, directory C:\output will contain a directory named BaseLibrary.Cpp containing the generated C++ source files and Cmake configuration files.

Now we want to use Cmake to generate makefile/project files. Let it be a Visual Studio 2017 x86 project file. In CMD we navigate to the C:\output\BaseLibrary.Cpp directory

```
>cd C:\output\BaseLibrary.Cpp
```

And run Cmake in configuration mode:

```
>Cmake --G "Visual Studio 15 2017"
```

And now we can build the sources using either Cmake or Visual Studio. Let us use Cmake:

```
>Cmake --build . --config Release
```


**The library is built.**


### Porting CommonLibrary ###

The second project in this example is located in CommonLibrary directory. CommonLibrary is a library project consisting of a single .cs source file BaseInterfaceSimplImpl.cs and a project file CommonLibrary.csproj. This project has a dependency on BaseLibrary project. This dependency has to be reflected in CommonLibrary project's configuration file. In our example this configuration file is pre-created, its name is CommonLibrary.porter.config and it is located in the project’s directory CommonLibrary. Let us have a closer look at the configuration file.

{{< highlight xml >}}
CommonLibrary.porter.config begins with an XML declaration, which specifies that the file contains an XML document  

Then goes the XML root element <porter> which is mandatory for Porter configuration XML document

    <porter>

Next, the default Porter configuration file is imported using <import> element. The default configuration will assign default values to all configuration options

    <import config="porter.config"/>

Also we need to import a configuration file include_map.config from ported BaseLibrary project that maps public types exported by CommonLibrary library to generated C++ header files in which these types are declared. include_map.config is generated by Porter for each project it ports. Thus, before porting CommonLibrary library project, BaseLibrary project has to be ported first so that Porter generates include_map.config. This is how include_map.config is included in CommonLibrary.porter.config:

    <import config="../../output/BaseLibrary.Cpp/include_map.config" />

Here ../../output is a directory that was passed as an output directory to Porter when BaseLibrary project was ported.

This example assumes that C# project CommonLibrary should be ported into C++ shared/dynamic library project, therefore we assign value ‘true’ to make_shared_lib option:

    <opt name="make_shared_lib" value="true" export_per_member="true"/>

Next, we want Porter to add some commands to the output CMakeLists.txt file. We do that by adding <cmake_commands> element to the configuration file containing raw Cmake commands

    <cmake_commands>
       <![CDATA[

The following commands set the output directory for the library’s binary by setting the corresponding properties on the target ${PROJECT_NAME}:

      set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY   "${CMAKE_CURRENT_SOURCE_DIR}/../bin")
      set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../bin")

Here ${PROJECT_NAME} is the name of the Cmake project which is equal to the name of the main Cmake executable target.

Because on “DLL-platforms” (i.e. Windows) a DLL shared library is considered by Cmake an executable entity and on “non-DLL-platforms” (i.e. Linux) a shared object is considered by Cmake a library, we set both RUNTIME_OUTPUT_DIRECTORY and LIBRARY_OUTPUT_DIRECTORY properties.

Then the <cmake_commands> element is closed:

      ]]>    
    </cmake_commands>

And last but not least, we need to tell Porter that CommonLibrary project depends on BaseLibrary library. We do it using <lib> element:

    <lib csname="BaseLibrary">
      <cmake_link_template>
        <![CDATA[
          find_package(BaseLibrary.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../BaseLibrary.Cpp" NO_DEFAULT_PATH)
          target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE BaseLibrary.Cpp)
        ]]>
       </cmake_link_template>
    </lib>

Here ${PROJECT_NAME}_dependencies is the name of the Cmake Interface Library target that is defined in the output CMakeLists.txt and is linked to main executable target ${PROJECT_NAME}. Thus libraries linked to ${PROJECT_NAME}_dependencies get automatically linked to ${PROJECT_NAME} target.

Finally the XML document is finished with closing tag of the root element <porter>:

    </porter>
{{< /highlight >}}

With the C# project at hand and configuration file ready, we can convert the project.

In order to covert CommonLibrary project we run CMD and navigate to the directory with porter binary:

```
>cd C:\CodePorting.Native_Cs2Cpp_19.4\bin\porter
```

And run Porter:

```
>CsToCppPorter.exe -c C:\ComplexNUnitTest\CommonLibrary\CommonLibrary.porter.config C:\ComplexNUnitTest\CommonLibrary\CommonLibrary.csproj C:\output
```

Porter will print some logs of the porting process to the console window and when it finishes porting, directory C:\output will contain a directory named CommonLibrary.Cpp containing the generated C++ source files and Cmake configuration files.

Now we want to use Cmake to generate makefile/project files. Let it be a Visual Studio 2017 x86 project file. In CMD we navigate to the C:\output\CommonLibrary.Cpp directory

```
>cd C:\output\CommonLibrary.Cpp
```

And run Cmake in configuration mode:

```
>Cmake --G "Visual Studio 15 2017"
```

And now we can build the sources using either Cmake or Visual Studio. Let us use Cmake:

```
>Cmake --build . --config Release
```

When build finishes, directory D:\output\bin\Release should contain a .dll file CommonLibrary.Cpp.dll. which has just been built from C++ sources.

### Porting LibraryA ###

The third project in this example is located in LibraryA directory. LibraryA is a library project consisting of a single .cs source file ClassAImpl.cs and a project file LibraryA.csproj. This project has a dependency on two previously ported projects BaseLibrary and CommonLibrary. These dependencies have to be reflected in LibraryA project's configuration file. In our example this configuration file is pre-created, its name is LibraryA.porter.config and it is located in the project’s directory LibraryA. Let’s have a closer look at this file.

{{< highlight xml >}}
LibraryA.porter.config begins with an XML declaration, which specifies that the file contains an XML document

Then goes the XML root element <porter> which is mandatory for Porter configuration XML document

    <porter>

Next, the default Porter configuration file is imported using <import> element. The default configuration will assign default values to all configuration options  

    <import config="porter.config"/>

Because LibraryA has dependency on BaseLibrary and CommonLibrary projects, we need to import configuration files include_map.config from both ported projects. This is how include_map.config files are included in LibraryA.porter.config:

    <import config="../../output/BaseLibrary.Cpp/include_map.config" />
    <import config="../../output/CommonLibrary.Cpp/include_map.config" />

Here ../../output is a directory that was passed as an output directory to Porter when BaseLibrary and CommonLibrary projects were ported.

And last but not least, we need to tell Porter that LibraryA project depends on BaseLibrary and CommonLibrary library. We do it using <lib> element  

    <lib name="CommonLibrary.Cpp" csname="CommonLibrary">
       <cmake_link_template>
         <![CDATA[
           find_package(CommonLibrary.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../CommonLibrary.Cpp" NO_DEFAULT_PATH)
           target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE CommonLibrary.Cpp)
         ]]>
      </cmake_link_template>
    </lib>
    <lib csname="BaseLibrary">
       <cmake_link_template>
         <![CDATA[
           find_package(BaseLibrary.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../BaseLibrary.Cpp" NO_DEFAULT_PATH)
           target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE BaseLibrary.Cpp)
         ]]>
      </cmake_link_template>
    </lib>

Finally the XML document is finished with closing tag of the root element <porter>:

    </porter>
{{< /highlight >}}


With C# project at hand and configuration file ready, we can convert the project.

In order to covert LibraryA project we run CMD and navigate to the directory with porter binary:

```
>cd C:\CodePorting.Native_Cs2Cpp_19.4\bin\porter
```

And run Porter:

```
>CsToCppPorter.exe -c C:\ComplexNUnitTest\LibraryA\LibraryA.porter.config C:\ComplexNUnitTest\LibraryA\LibraryA.csproj C:\output
```

Porter will print some logs of the porting process to the console window and when it finishes porting, directory C:\output will contain a directory named LibraryA.Cpp containing the generated C++ source files and Cmake configuration files.

Now we want to use Cmake to generate makefile/project files. Let it be a Visual Studio 2017 x86 project file. In CMD we navigate to the C:\output\LibraryA.Cpp directory

```
>cd C:\output\LibraryA.Cpp
```

And run Cmake in configuration mode:

```
>Cmake --G "Visual Studio 15 2017"
```

And now we can build the sources using either Cmake or Visual Studio. Let us use Cmake:

```
>Cmake --build . --config Release
```

**The library is built.**

### Porting LibraryB ###

The fourth project in this example is located in LibraryB directory. LibraryB is a library project consisting of a single .cs source file ClassBImpl.cs and a project file LibraryB.csproj. This project has a dependency on two previously ported projects BaseLibrary and CommonLibrary. These dependencies have to be reflected in LibraryB project’s configuration file. In our example this configuration file is pre-created, its name is LibraryB.porter.config and it is located in the project’s directory LibraryB. Let us have a closer look at LibraryB's configuration file.

{{< highlight xml >}}
LibraryB’s configuration file is much the same as LibraryA’s one described above. One difference is that because this example assumes that LibraryB should be built into a shared dynamic library, this has to be explicitly stated in the configuration file

    <opt name="make_shared_lib" value="true" export_per_member="true"/>

And another difference is that we want to set the output directory for the library binary file just like we did it in ComomnLibary’s configuration file described above

    <cmake_commands>
      <![CDATA[
        set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../bin")
        set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../bin")
      ]]>
    </cmake_commands>
{{< /highlight >}}

With C# project at hand and configuration file ready, we can convert the project.

In order to covert LibraryB project we run CMD and navigate to the directory with porter binary:

```
>cd C:\CodePorting.Native_Cs2Cpp_19.4\bin\porter
```

And run Porter:

```
>CsToCppPorter.exe -c C:\ComplexNUnitTest\LibraryB\LibraryB.porter.config C:\ComplexNUnitTest\LibraryB\LibraryB.csproj C:\output
```

Porter will print some logs of the porting process to the console window and when it finishes porting, directory C:\output will contain a directory named LibraryB.Cpp containing the generated C++ source files and Cmake configuration files.

Now we want to use Cmake to generate makefile/project files. Let it be a Visual Studio 2017 x86 project file. In CMD we navigate to the C:\output\LibraryB.Cpp directory

```
>cd C:\output\LibraryB.Cpp
```

And run Cmake in configuration mode:

```
>Cmake --G "Visual Studio 15 2017"
```

And now we can build the sources using either Cmake or Visual Studio. Let us use Cmake:

```
>Cmake --build . --config Release
```

When build finishes, directory D:\output\bin\Release should contain just built .dll file LibraryB.Cpp.dll along with previously built CommonLibrary.Cpp.dll.

### Porting ComplexNUnitTest ###

The last project in this example is located in ComplexNUnitTest directory. ComplexNUnitTest is a library project that contains NUnit tests. Porter translates C# NUnit library projects into C++ executable projets. ComplexNUnitTest project consists of a single .cs source file ComplexTest.cs and a project file ComplexNUnitTest.csproj. This project has a dependency on all four previously ported projects – BaseLibrary, CommonLibrary, LibraryA and LibraryB. These dependencies have to be reflected in the ComplexNUnitTest project's configuration file. In our example this configuration file is pre-created, its name is ComplexNUnitTest.porter.config and it is located in the project’s directory ComplexNUnitTest. Let us have a closer look at the configuration file.

{{< highlight xml >}}
ComplexNUnitTest.porter.config begins with an XML declaration, which specifies that the file contains an XML document

Then goes the XML root element <porter> which is mandatory for Porter configuration XML document

  <porter>

Next, the default Porter configuration file is imported using <import> element. The default configuration will assign default values to all configuration options

  <import config="porter.config"/>

Also we import include_map.config files from all four previously ported library projects

  <import config="../../output/LibraryA.Cpp/include_map.config" />
  <import config="../../output/LibraryB.Cpp/include_map.config" />
  <import config="../../output/BaseLibrary.Cpp/include_map.config" />
  <import config="../../output/CommonLibrary.Cpp/include_map.config" />

Next, we want Porter to add some commands to the output CMakeLists.txt. We do that by adding <cmake_commands> element to the configuration file containing raw Cmake commands

  <cmake_commands>

    <![CDATA[   

The first command sets the output directory for the executable binary by setting the corresponding property on the target ${PROJECT_NAME}_gtest

      set_target_properties(${PROJECT_NAME}_gtest PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../bin")

Here ${PROJECT_NAME} is the name of the Cmake project and ${PROJECT_NAME}_gtest is the name of the main Cmake executable target.

Then the <cmake_commands> element is closed

      ]]>
     </cmake_commands>

Then, we need to tell Porter that ComplexNUnitTest project depends on four libraries. We do it using <lib> element:

    <lib name="BaseLibrary.Cpp" csname="BaseLibrary">
      <cmake_link_template>
        <![CDATA[
          find_package(BaseLibrary.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../BaseLibrary.Cpp" NO_DEFAULT_PATH)
          target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE BaseLibrary.Cpp)
        ]]>
     </cmake_link_template>
    </lib>

    <lib name="CommonLibrary.Cpp" csname="CommonLibrary">
       <cmake_link_template>
         <![CDATA[
           find_package(CommonLibrary.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../CommonLibrary.Cpp" NO_DEFAULT_PATH)
           target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE CommonLibrary.Cpp)
         ]]>
      </cmake_link_template>
    </lib>

    <lib name="LibraryA.Cpp" csname="LibraryA">
      <cmake_link_template>
        <![CDATA[
          find_package(LibraryA.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../LibraryA.Cpp" NO_DEFAULT_PATH)
          target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE LibraryA.Cpp)           
        ]]>
      </cmake_link_template>
    </lib>

    <lib name="LibraryB.Cpp" csname="LibraryB">
       <cmake_link_template>
        <![CDATA[
          find_package(LibraryB.Cpp REQUIRED CONFIG PATHS "${CMAKE_CURRENT_SOURCE_DIR}/../LibraryB.Cpp" NO_DEFAULT_PATH)
          target_link_libraries(${PROJECT_NAME}_dependencies INTERFACE LibraryB.Cpp)
        ]]>
      </cmake_link_template>
    </lib>

Here ${PROJECT_NAME}_dependencies is the name of the Cmake Interface Library target that is defined in the output CMakeLists.txt file and is linked to main executable target ${PROJECT_NAME}_gtest. Thus libraries linked to ${PROJECT_NAME}_dependencies get automatically linked to ${PROJECT_NAME}_gtest target.

Finally the XML document is finished with closing tag of the root element <porter>

</porter>
{{< /highlight >}}

With the C# project at hand and configuration file ready, we can convert the project.

In order to covert ComplexNUnitTest project we run CMD and navigate to the directory with porter binary:

```
>cd C:\CodePorting.Native_Cs2Cpp_19.4\bin\porter
```

And run Porter:

```
>CsToCppPorter.exe -c C:\ComplexNUnitTest\ComplexNUnitTest\ComplexNUnitTest.porter.config C:\ComplexNUnitTest\ComplexNUnitTest\ComplexNUnitTest.csproj C:\output
```

Porter will print some logs of the porting process to the console window and when it finishes porting, directory C:\output will contain a directory named ComplexNUnitTest.Cpp containing the generated C++ source files and Cmake configuration files.

Now we want to use Cmake to generate makefile/project files. Let it be a Visual Studio 2017 x86 project file. In CMD we navigate to the C:\output\ComplexNUnitTest.Cpp directory

```
>cd C:\output\ComplexNUnitTest.Cpp
```

And run Cmake in configuration mode:

```
>Cmake --G "Visual Studio 15 2017"
```

And now we can build the sources using either Cmake or Visual Studio. Let us use Cmake:

```
>Cmake --build . --config Release
```


When build finishes, directory D:\output\bin\Release should contain four files: CommonLibrary.dll, LibraryB.dll, ComplexNUnitTest.Cpp_gtest.exe, which has just been built from sources, and aspose_cpp_vc140.dll, which was copied from Porter installation directory during a post-build step. When we run ComplexNUnitTest.Cpp_gtest.exe it executes tests and prints results to the console window. The tests ComplexNUnitTest.Cpp_gtest.exe executes are similar to those in original C# project.