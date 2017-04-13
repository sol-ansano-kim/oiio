import excons
import re
import sys
import excons
import os
import glob


major = 1
minor = 7
patch = 12


env = excons.MakeBaseEnv()
staticlib = (excons.GetArgument("oiio-static", 0, int) != 0)
out_basedir = excons.OutputBaseDirectory()
out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

prjs = []
oiio_opts = {}
ocio_overrides = {}
png_overrides = {}
tiff_overrides = {}
libraw_overrides = {}
freetype_overrides = {}
jpeg_overrides = {}
openexr_overrides = {}
oiio_dependecies = []
extra_libs = []
extra_includes = []

# build options
oiio_opts["LINKSTATIC"] = True
oiio_opts["USE_fPIC"] = True
oiio_opts["BUILDSTATIC"] = staticlib
oiio_opts["NOTHREADS"] = False
oiio_opts["OIIO_THREAD_ALLOW_DCLP"] = True
oiio_opts["SOVERSION"] = "%s.%s" % (major, minor)
oiio_opts["USE_CPP11"] = True
oiio_opts["USE_CPP14"] = False
oiio_opts["USE_LIBCPLUSPLUS"] = False
oiio_opts["USE_CCACHE"] = False

# python
oiio_opts["USE_PYTHON"] = True
oiio_opts["USE_PYTHON3"] = False
oiio_opts["PYTHON_VERSION"] = 2.7
oiio_opts["PYTHON3_VERSION"] = 3.2
oiio_opts["PYLIB_INCLUDE_SONAME"] = False
oiio_opts["PYLIB_LIB_PREFIX"] = False

## addtional
oiio_opts["BOOST_ROOT"] = excons.GetArgument("with-boost", "", str)
oiio_opts["USE_FIELD3D"] = False
oiio_opts["USE_JPEGTURBO"] = True
oiio_opts["USE_OPENJPEG"] = True
oiio_opts["USE_FREETYPE"] = True
oiio_opts["USE_LIBRAW"] = True
oiio_opts["USE_OCIO"] = True
oiio_opts["USE_FFMPEG"] = False

## extra args
oiio_opts["EXTRA_CPP_ARGS"] = ""
oiio_opts["EXTRA_DSO_LINK_ARGS"] = ""

## jpeg build option
jpeg_overrides["libjpeg-jpeg8"] = 1


if sys.platform == "darwin":
    oiio_opts["EXTRA_CPP_ARGS"] = " -Wno-deprecated-declarations"


# zlib
def ZlibName(static):
    return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
    return ([] if  static else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "zlib", libnameFunc=ZlibName, definesFunc=ZlibDefines)
if not rv:
    excons.PrintOnce("Build zlib from sources ...")
    excons.Call("zlib", imp=["ZlibName", "ZlibPath"])    
    z_static = excons.GetArgument("zlib-static", 1, int)
    z_path = ZlibPath(static=z_static)
    zlib_outputs = [excons.cmake.OutputsCachePath("zlib")]

    oiio_opts["ZLIB_INCLUDE_DIR"] = out_incdir
    oiio_opts["ZLIB_LIBRARY"] = z_path

    base = os.path.dirname(os.path.dirname(z_path))
    name = ZlibName(static=z_static)

    png_overrides["with-zlib"] = base    
    png_overrides["zlib-static"] = z_static
    png_overrides["zlib-name"] = name
    tiff_overrides["with-zlib"] = base
    tiff_overrides["zlib-static"] = z_static
    tiff_overrides["zlib-name"] = name
    freetype_overrides["with-zlib"] = base
    freetype_overrides["zlib-static"] = z_static
    freetype_overrides["zlib-name"] = name
    openexr_overrides["with-zlib"] = base
    openexr_overrides["zlib-static"] = z_static
    openexr_overrides["zlib-name"] = name
else:
    zlib_outputs = []

# jbig
rv = excons.cmake.ExternalLibRequire(oiio_opts, "jbig")
if not rv:
    excons.PrintOnce("Build jbig from sources ...")
    excons.Call("jbigkit", imp=["JbigName", "JbigPath"])
    jbig_path = JbigPath()
    jbig_outputs = [jbig_path, out_incdir + "/jbig.h", out_incdir + "/jbig85.h", out_incdir + "/jbig_ar.h"]

    tiff_overrides["with-jbig"] = os.path.dirname(os.path.dirname(jbig_path))
    tiff_overrides["jbig-static"] = 1
    tiff_overrides["jbig-name"] = JbigName()
else:
    jbig_outputs = []


# lcms2
rv = excons.cmake.ExternalLibRequire(oiio_opts, "lcms2", varPrefix="LCMS2_")
if not rv:
    excons.PrintOnce("Build lcms from sources ...")
    excons.Call("Little-cms", imp=["LCMS2Name", "LCMS2Path"])
    lcms2_path = LCMS2Path()
    lcms2_outputs = [lcms2_path]

    oiio_opts["LCMS2_LIBRARY"] = lcms2_path
    oiio_opts["LCMS2_INCLUDE_DIR"] = out_incdir

    lcms2_static = excons.GetArgument("lcms2-static", 1, int)
    base = os.path.dirname(os.path.dirname(lcms2_path))
    name = LCMS2Name()

    libraw_overrides["with-lcms2"] = base
    libraw_overrides["lcms2-static"] = lcms2_static
    libraw_overrides["lcms2-name"] = name
    libraw_overrides["libraw-with-lcms2"] = 1

    ocio_overrides["with-lcms2"] = base
    ocio_overrides["lcms2-static"] = lcms2_static
    ocio_overrides["lcms2-name"] = name
    
else:
    lcms2_outputs = []


# jpeg
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libjpeg", varPrefix="JPEG_")
if not rv:
    excons.PrintOnce("Build libjpeg-turbo from sources ...")
    excons.Call("libjpeg-turbo", overrides=jpeg_overrides, imp=["LibjpegName LibjpegPath"])
    oiio_opts["JPEG_INCLUDE_DIR"] = out_incdir
    oiio_opts["JPEG_LIBRARY"] = LibjpegPath(static=excons.GetArgument("libjpeg-static", 1, int))

    jpeg_static = excons.GetArgument("libjpeg-static", 1, int)
    jpeg_path = LibjpegPath(static=jpeg_static)
    if sys.platform == "win32":
        jpeg_outputs = [excons.cmake.OutputsCachePath("libjpeg")]
    else:
        jpeg_outputs = [excons.automake.OutputsCachePath("libjpeg")]

    base = os.path.dirname(os.path.dirname(jpeg_path))
    name = LibjpegName(jpeg_static)

    tiff_overrides["with-libjpeg"] = base
    tiff_overrides["libjpeg-static"] = jpeg_static
    tiff_overrides["libjpeg-name"] = name
    libraw_overrides["with-libjpeg"] = base
    libraw_overrides["libjpeg-static"] = jpeg_static
    libraw_overrides["libjpeg-name"] = name
else:
    jpeg_outputs = []

oiio_dependecies += jpeg_outputs

# png
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libpng", varPrefix="PNG_")
if not rv:
    excons.PrintOnce("Build libpng from sources ...")
    excons.cmake.AddConfigureDependencies("libpng", zlib_outputs)
    excons.Call("libpng", overrides=png_overrides, imp=["LibpngName", "LibpngPath"])
    oiio_opts["PNG_INCLUDE_DIR"] = out_incdir
    oiio_opts["PNG_LIBRARY"] = LibpngPath(static=excons.GetArgument("libpng-static", 1, int))
    libpng_outputs = [excons.cmake.OutputsCachePath("libpng")]

    png_static = excons.GetArgument("libpng-static", 1, int)
    base = os.path.dirname(os.path.dirname(LibpngPath(png_static)))
    name = LibpngName(png_static)
    
    freetype_overrides["with-libpng"] = base
    freetype_overrides["libpng-static"] = png_static
    freetype_overrides["libpng-name"] = name

else:
    libpng_outputs = []

oiio_dependecies += libpng_outputs

# tiff
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libtiff", varPrefix="TIFF_")
if not rv:
    excons.PrintOnce("Build libtiff from sources ...")
    excons.cmake.AddConfigureDependencies("libtiff", jbig_outputs + zlib_outputs + jpeg_outputs)

    tiff_outputs = [excons.cmake.OutputsCachePath("libtiff")]
    excons.Call("libtiff", overrides=tiff_overrides, imp=["LibtiffPath"])
    oiio_opts["TIFF_INCLUDE_DIR"] = out_incdir
    oiio_opts["TIFF_LIBRARY"] = LibtiffPath()
else:
    tiff_outputs = []

extra_libs.append(JbigPath())
oiio_dependecies += tiff_outputs


# openjpeg
rv = excons.cmake.ExternalLibRequire(oiio_opts, "openjpeg")
if not rv:
    excons.PrintOnce("Build openjpeg from sources ...")
    excons.Call("openjpeg", imp=["OpenjpegPath"])
    oiio_opts["OPENJPEG_INCLUDE_DIR"] = out_incdir
    oiio_opts["OPENJPEG_LIBRARY"] = OpenjpegPath()
    openjpeg_outputs = [excons.cmake.OutputsCachePath("openjpeg")]
else:
    openjpeg_outputs = []

oiio_dependecies += openjpeg_outputs


# libraw
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libraw")
if not rv:
    excons.PrintOnce("Build libraw from sources ...")
    excons.Call("LibRaw", overrides=libraw_overrides, imp=["LibrawPath", "LibrawName"])
    oiio_opts["LIBRAW_INCLUDE_DIR"] = out_incdir
    oiio_opts["LIBRAW_LIBRARY"] = LibrawPath()
    libraw_outputs = [LibrawPath()]
else:
    libraw_outputs = []

oiio_dependecies += libraw_outputs


# bzip2
def Bzip2Libname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

def Bzip2Defines(static):
    return ([] if static else ["BZ_DLL"])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "bz2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines, varPrefix="BZIP2_")
if not rv:
    excons.PrintOnce("Build bzip2 from sources ...")
    excons.Call("bzip2", imp=["BZ2Name", "BZ2Path"])
    bz2_static = excons.GetArgument("bz2-static", 1, int)

    freetype_overrides["with-bz2"] = base
    freetype_overrides["bz2-static"] = bz2_static
    freetype_overrides["bz2-name"] = name

    bzip2_outputs = [BZ2Path()]
else:
    bzip2_outputs = []


# freetype
rv = excons.cmake.ExternalLibRequire(oiio_opts, "freetype")
if not rv:
    excons.PrintOnce("Build freetype from sources ...")
    excons.cmake.AddConfigureDependencies("freetype", zlib_outputs + libpng_outputs + bzip2_outputs)
    excons.Call("freetype", overrides=freetype_overrides, imp=["FreetypeName", "FreetypePath"])
    oiio_opts["FREETYPE_INCLUDE_DIR"] = out_incdir
    oiio_opts["FREETYPE_LIBRARY"] = FreetypePath()
    freetype_outputs = [excons.cmake.OutputsCachePath("freetype")]
else:
    freetype_outputs = []

oiio_dependecies += freetype_outputs


# ocio
rv = excons.cmake.ExternalLibRequire(oiio_opts, "OpenColorIO")
if not rv:
    excons.PrintOnce("Build OpenColorIO from sources ...")
    excons.Call("OpenColorIO", overrides=ocio_overrides, imp=["OCIOPath"])
    ocio_static = excons.GetArgument("ocio-static", 1, int) != 0
    ocio_outputs = [OCIOPath(ocio_static)]
    # TODO : fix oiio cmake config
    oiio_opts["OCIO_INCLUDE_DIR"] = out_incdir
    # oiio_opts["OCIO_LIBRARY"] = OCIOPath(ocio_static)
    oiio_opts["OCIO_LIBRARIES"] = OCIOPath(ocio_static)
    ext = ".lib" if sys.platform == "win32" else (".a" if ocio_static else ".so")
    oiio_opts["LCMS2_LIBRARY"] = LCMS2Path()
    oiio_opts["YAML_LIBRARY"] = out_libdir + "/libyaml-cpp" + ext
    oiio_opts["TINYXML_LIBRARY"] = out_libdir + "/libtinyxml" + ext
else:
    ocio_outputs = []

oiio_dependecies += ocio_outputs


# opexnexr
openexr_h_patterns = ["openexr/IlmBase/Half/half.h",
                      "openexr/IlmBase/Half/halfExport.h",
                      "openexr/IlmBase/Half/halfFunction.h",
                      "openexr/IlmBase/Half/halfLimits.h",
                      "openexr/IlmBase/Iex/*.h",
                      "openexr/IlmBase/IexMath/*.h",
                      "openexr/IlmBase/Imath/*.h",
                      "openexr/IlmBase/IlmThread/*.h",
                      "openexr/OpenEXR/IlmImfUtil/*.h"]


rv = excons.cmake.ExternalLibRequire(oiio_opts, "openexr")
if not rv:
    excons.PrintOnce("Build openexr from sources ...")
    excons.Call("openexr", overrides=openexr_overrides)
    openexr_static = (excons.GetArgument("openexr-static", 1, int) != 0)
    openexr_lib_suffx = excons.GetArgument("openexr-suffix", "-2_2")
    openexr_static_suffic = excons.GetArgument("openexr-static-suffix", "_s")
    ext = ".lib" if sys.platform == "win32" else (".a" if openexr_static else ".so")

    openexr_sf = openexr_lib_suffx + (openexr_static_suffic if openexr_static else "") + ext
    iex = out_libdir + "/libIex" + openexr_sf
    half = out_libdir + "/libHalf" + openexr_sf
    imf = out_libdir + "/libIlmImf" + openexr_sf
    ilmt = out_libdir + "/libIlmThread" + openexr_sf
    imath = out_libdir + "/libIexMath" + openexr_sf

    oiio_opts["OPENEXR_HOME"] = out_basedir
    oiio_opts["OPENEXR_INCLUDE_DIR"] = out_incdir
    oiio_opts["OPENEXR_IEX_LIBRARY"] = iex
    oiio_opts["OPENEXR_HALF_LIBRARY"] = half
    oiio_opts["OPENEXR_ILMIMF_LIBRARY"] = imf
    oiio_opts["OPENEXR_ILMTHREAD_LIBRARY"] = ilmt
    oiio_opts["OPENEXR_IMATH_LIBRARY"] = imath

    openexr_h = []
    openexr_h_deps = []
    for p in openexr_h_patterns:
        openexr_h += glob.glob(p)
    openexr_h += filter(lambda x: os.path.splitext(os.path.basename(x))[0] not in ["b44ExpLogTable", "dwaLookups"], glob.glob("openexr/OpenEXR/IlmImf/*.h"))

    openexr_h_deps = map(lambda x: "release/include/OpenEXR/" + os.path.basename(x), openexr_h)
    openexr_h_deps += ["release/include/OpenEXR/" + "IlmBaseConfig.h"]
    openexr_h_deps += ["release/include/OpenEXR/" + "OpenEXRConfig.h"]

    openexr_outputs = [iex, half, imf, ilmt, imath] + openexr_h_deps
else:
    openexr_outputs = []


oiio_dependecies += openexr_outputs


# oiio build
oiio_opts["EXTERNAL_LIBS"] = ";".join(extra_libs)
oiio_opts["EXTERNAL_INCLUDE_DIRS"] = ";".join(extra_includes)

prjs.append({"name": "oiio",
             "type": "cmake",
             "cmake-opts": oiio_opts,
             "cmake-cfgs": excons.CollectFiles(["src"], patterns=["CMakeLists.txt"], recursive=True) + ["CMakeLists.txt"] + oiio_dependecies,
             "cmake-srcs": excons.CollectFiles(["src"], patterns=["*.cpp"], recursive=True)})


excons.AddHelpOptions(oiio="""OpenImageIO OPTIONS
    oiio-static=0|1   :  Toggle between static and shared library build [0]
    """)


excons.DeclareTargets(env, prjs)


def OiioName(static=False):
    libname = "OpenImageIO"
    if sys.platform == "win32" and static:
        libname += "-static"
    return libname


def OiioPath(static=False):
    name = OiioName(static=static)
    if sys.platform == "win32":
        libname = name + ".lib"
    else:
        libname = "lib" + name + (".a" if static else excons.SharedLibraryLinkExt())

    return excons.OutputBaseDirectory() + "/lib/" + libname


def RequireOiio(env, static=False):
    env.Append(CPPPATH=[excons.OutputBaseDirectory() + "/include"])
    env.Append(LIBPATH=[excons.OutputBaseDirectory() + "/lib"])

    excons.Link(env, OiioName(static=static), static=static, force=True, silent=True)


Default("oiio")
Export("OiioName OiioPath RequireOiio")
