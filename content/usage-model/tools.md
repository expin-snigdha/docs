    ## Prerequisites
    1. [Shell](shell link)

    ## Overview

    Modules, built on the [Environment Modules](https://modules.readthedocs.io/en/stable/index.html) open-source project, provide a way to manage open-source, internal, and vendor-provided tools.`module` command enables users to interact with the modules system, allowing for easy loading, unloading, and swapping of tool versions, as well as bundling tools for specific purposes or projects.

    ## Tools

    Each tool version is managed through a module-file released by the infrastructure team. Loading a module (`module load <module-file>`) in a shell modifies environment variables (see below). Note that modules system adds several other environment variables which are not relevant within the scope of this document.

    * `PATH`: Prepends the tool's bin directory, usually `<tool_root>/<version>/bin`, ensuring that the executable files for the tool are accessible from the command line.
    * `MANPATH`: Prepends the tool's man pages directory, usually `<tool_root>/<version>/share/man`, making manual pages for the tool available for the man command.
    * `LD_LIBRARY_PATH`: Prepends the tool's lib directory, usually `<tool_root>/<version>/lib64`, if available, allowing the dynamic linker to find the tool’s shared libraries.
    * `CMAKE_PREFIX_PATH`: Prepends the tool's cmake search path, usually `<tool_root>/<version>`, enabling cmake to find the tool's components using find_package.
    * `PKG_CONFIG_PATH`: Prepends the tool's pkgconfig path, usually `<tool_root>/<version>/lib64/pkgconfig`, where `.pc` files are present. This enables both cmake and GNU pkg-config in locating the tool’s libraries, dependencies and other components.
    * `PYTHONPATH`: If applicable, prepends the tool or package's Python modules directory, allowing Python scripts to find and import modules made available.
    * `CPATH`: If applicable, prepends the tool’s include directories, allowing C and C++ compilers to find header files for the tool. [WIP]
    * `LIBRARY_PATH`: If applicable, prepends the tool’s library directories for static linking, ensuring that the linker can find the tool’s static libraries. [WIP]

    ### Module file

    Here is the module file for `gcc 13.2.0`:
    ```
    #%Module
    set __tool_root "/auto/tools/installs"
    set __tool_name "gcc"
    set __tool_version "13.2.0"
    set __tool_prefix $__tool_root/$__tool_name/$__tool_version

    proc ModulesHelp {} {
        puts stderr "loads tool=$_tool_name version=$__tool_version from $__tool_root/$__tool_name/$__tool_version"
        puts stderr {}
    }

    module-whatis "loads tool=$__tool_name version=$__tool_version from $__tool_root/$__tool_name/$__tool_version"

    prepend-path PATH              $__tool_prefix/bin
    prepend-path MANPATH           $__tool_prefix/share/man
    prepend-path LD_LIBRARY_PATH   $__tool_prefix/lib64
    prepend-path CPATH             $__tool_prefix/include
    prepend-path CMAKE_PREFIX_PATH $__tool_prefix

    unset __tool_root
    unset __tool_name
    unset __tool_version
    unset __tool_prefix

    ```


    ## Tool bundles

    Tool bundle itself is a module which loads a set of modules. This mechanism allows a set of dependent tools and required version of each, say for a project, to be loaded. An example bundle is `sw.v0.1` all the bundles are present in `sw` subdirectory of the module hierarchy. 
    a
    Here is a tool bundle file for `sw.v0.1`:

    ```
    #%Module

    proc ModulesHelp {} {
        puts stderr "loads sw.v0.1 tool bundle"
        puts stderr {}
    }

    module-whatis "loads sw.v0.1 tool bundle"

    module load cmake/3.29.5
    module load gcc/13.2.0
    module load python/3.11.0
    ```


    ### Composition and Management

    The composition, versioning, and approvals for changes in tool bundles are managed by the respective project owners. 

    ## Using module command

    1. Listing loaded tools:


    2. Listing all available tools:


    3. Displaying a module's details:


    4. Loading a module (tool):
    ```
    module load <module_name>/<version>
    module load gcc/13.02.1
    ```

    5. Swapping a module, say you'd like try gcc 11 instead of gcc 13:
    ```
    module swap <old_module>/<old_version> <new_module>/<new_version>
    module swap cmake/3.29.5 cmake/4.1.0
    ```

    6. Purging (unloading) all modules:
    ```
    module purge
    ```

    7. Loading a tool bundle
    ```
    module load <bundle_name>
    module load sw.v0.1
    ```
    8. Listing available tool bundles


    ## Open source tools

    ### Availability
    For open source tools, each tool is downloaded, compiled on our machines and released (module avail will eventually show it).

    ### Installation path
    All the open source tools are installed to /auto/tools directory on a infra machine. 

    ### Requesting new version or new tool


    ## Vendor tools

    ### Availability
    Hardware team and infrastructure team work together to install the EDA tools and license manager. Hardware team reads the tool setup and user guides and provides necessary settings for each tool. A module-file is then created.

    ### Installation path

    ### Requesting new version or new tool

