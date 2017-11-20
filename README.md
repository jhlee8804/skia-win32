# skia-build-dll
Build google skia library as dll.



##### 1. Install depot_tools (Package of scripts)
    https://www.chromium.org/developers/how-tos/depottools


##### 2. Check out the Skia source code (https://skia.org/user/quick/windows)
    $ git clone https://skia.googlesource.com/skia.git skia
    $ cd skia
  
  
##### 3. GYP Generators on Windows (https://skia.org/user/quick/windows)
We use the open-source Gyp tool to generate Visual Studio projects (and analogous build scripts on other platforms) 
from our multi-platform “gyp” files.
  
Three Gyp generators are used on Windows:

    - ninja: Run ninja yourself, without VisualStudio project files,
    - msvs-ninja: Develop from a fully-integrated Visual Studio. Gyp generates 
                  Visual-Studio-compatible project files that still ultimately build using ninja
    - msvs: Use Visual Studio’s own (slower) build system
    
To choose which ones to use, set the GYP_GENERATORS environment variable to a comma-delimited list of generators before running sync-and-gyp. The default value for GYP_GENERATORS is ninja,msvs-ninja. For example to enable the ninja and msvs generators:


##### 4.  Modify Build Params in /bin/sync-and-gyp
    // add '-Dskia_shared_lib=1'
    //subprocess.call(['python', './gyp_skia',], env=env)
    subprocess.call(['python', './gyp_skia', '-Dskia_shared_lib=1'], env=env)
    

##### 5. Modify RuntimeLibrary Option in /gyp/common_conditions.gypi.
    [ 'skia_os == "win"',
        ...
        'configurations': {
            'Debug': {
                'msvs_settings': {
                    'VCCLCompilerTool': {
                        // RuntimeLibrary 3 -> 1
                        #'RuntimeLibrary': '3',         # rtMultiThreadedDebugDLL (/MDd)
                        'RuntimeLibrary': '1',  # /MTd
                        ...
                    },
                ...
                },
            ...
            },
            'Release': {
                'msvs_settings': {
                    'VCCLCompilerTool': {
                        // RuntimeLibrary 2 -> 0
                        #'RuntimeLibrary': '2',              # rtMultiThreadedDLL (/MD)
                        'RuntimeLibrary': '0',  # /MT
                        ...
                    },
                ...
                },
            ...
            }
        },
        'conditions' : [
            ...
            // add follow coditions
            [ 'skia_shared_lib', {
                'defines': [
                    'SKIA_DLL',
                    'SKIA_IMPLEMENTATION=1',
                ],
            }],
            ...
        ],


##### 6. Turn Off 'skia_warnings_as_error' in /gyp/common_variables.gypi
    'conditions': [
        [ 'skia_os in ["mac", "linux", "freebsd", "openbsd", "solaris", "android", "win"] '
                'and skia_android_framework == 0', {
            //'skia_warnings_as_errors%': 1,
            'skia_warnings_as_errors%': 0,
        }, {
            'skia_warnings_as_errors%': 0,
        }],

##### 7. Setting Enviroment Variables in Windows CMD.EXE (https://skia.org/user/tips#gypdefines)
    $ cd %SKIA_CHECKOUT_DIR%
    $ SET "GYP_GENERATORS=msvs"
    $ python bin/sync-and-gyp
    $ SET "GYP_GENERATORS="
    

##### 8. Set 'True' to 'Linker - General - Use Library Dependency Inputs'
    skia_lib


##### 9. Editing follow codes in "include/core/SkPreConfig."
    #if defined(SKIA_DLL)
        //#if defined(WIN32)
        #if defined(SK_BUILD_FOR_WIN32)                                 <<<<<<<<<<<<<<<<<<<
            #if SKIA_IMPLEMENTATION
                #define SK_API __declspec(dllexport)
            #else
                #define SK_API __declspec(dllimport)
            #endif
        #else
            #define SK_API __attribute__((visibility("default")))
        #endif
    #else
        #define SK_API
    #endif

    
##### 10. Build the skia_lib project !!!
    
