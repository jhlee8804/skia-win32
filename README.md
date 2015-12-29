# skia-build-dll
Build google skia library as dll.



#####1. Install depot_tools (Package of scripts)
    >https://www.chromium.org/developers/how-tos/depottools


#####2. Check out the Skia source code (https://skia.org/user/quick/windows)
    >git clone https://skia.googlesource.com/skia.git skia
    >cd skia
  
  
#####3. GYP Generators on Windows (https://skia.org/user/quick/windows)
We use the open-source Gyp tool to generate Visual Studio projects (and analogous build scripts on other platforms) 
from our multi-platform “gyp” files.
  
Three Gyp generators are used on Windows:

    - ninja: Run ninja yourself, without VisualStudio project files,
    - msvs-ninja: Develop from a fully-integrated Visual Studio. Gyp generates 
                  Visual-Studio-compatible project files that still ultimately build using ninja
    - msvs: Use Visual Studio’s own (slower) build system
    
To choose which ones to use, set the GYP_GENERATORS environment variable to a comma-delimited list of generators before running sync-and-gyp. The default value for GYP_GENERATORS is ninja,msvs-ninja. For example to enable the ninja and msvs generators:


#####4. Setting Enviroment Variables in Windows CMD.EXE (https://skia.org/user/tips#gypdefines)
    >cd %SKIA_CHECKOUT_DIR%
    >SET "GYP_GENERATORS=msvs"
    >python gyp_skia -Dskia_shared_lib=1
    >SET "GYP_GENERATORS="
    

#####5. Add macros('SKIA_IMPLEMENTATION, SKIA_DLL') to follow projects.
    >codec
    >core
    >effects
    >ports
    >skgpu (if set GYP_DEFINES='skia_gpu=1')
    >skia_lib


#####6. Change Project Options.
    C/C++ - Runtime Library: "Multi-threaded Debug (/MTd), Multi-threaded (/MT)"
    C/C++ - Treat Warnings As Errors: NO(/WX-)
    
    >codec
    >codec_android
    >core
    >effect
    >images
    >libetc1
    >libSkKTX
    >opts
    >opts_avx
    >opts_sse41
    >opts_ssse3
    >ports
    >skgpu
    >skia_lib
    >sfnt
    >utils


#####6. Editing follow codes in "SkPreConfig."
    #if defined(SKIA_DLL)
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

    
#####7. Build the skia_lib project !!!
    
