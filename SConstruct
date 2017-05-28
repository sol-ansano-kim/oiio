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
python_dir = excons.GetArgument("oiio-python-dir", "", str)
python_ver = excons.GetArgument("oiio-python-ver", "2.7", str)

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
oiio_opts["LINKSTATIC"] = 1
oiio_opts["USE_fPIC"] = 1
oiio_opts["BUILDSTATIC"] = (1 if staticlib else 0)
oiio_opts["NOTHREADS"] = 0
oiio_opts["OIIO_THREAD_ALLOW_DCLP"] = 1
oiio_opts["SOVERSION"] = "%s.%s" % (major, minor)
oiio_opts["USE_CPP11"] = 1
oiio_opts["USE_CPP14"] = 0
oiio_opts["USE_LIBCPLUSPLUS"] = 0
oiio_opts["USE_CCACHE"] = 0
oiio_opts["HIDE_SYMBOLS"] = 1

# python
oiio_opts["PYLIB_INCLUDE_SONAME"] = 0
oiio_opts["PYLIB_LIB_PREFIX"] = 0

if int(python_ver.split(".")[0]) == 2:
    oiio_opts["PYTHON_VERSION"] = python_ver
    oiio_opts["USE_PYTHON"] = 1
    oiio_opts["USE_PYTHON3"] = 0
else:
    oiio_opts["PYTHON3_VERSION"] = python_ver
    oiio_opts["USE_PYTHON"] = 0
    oiio_opts["USE_PYTHON3"] = 1

if python_dir:
    if sys.platform == "win32":
        oiio_opts["PYTHON_LIBRARY"] = "%s/libs/python%s.lib" % (python_dir, python_ver.replace(".", ""))
        oiio_opts["PYTHON_EXECUTABLE"] = "%s/python.exe" % (python_dir)
    else:
        oiio_opts["PYTHON_LIBRARY"] = "%s/libs/libpython%s.so" % (python_dir, python_ver)
        oiio_opts["PYTHON_EXECUTABLE"] = "%s/python" % (python_dir)

    oiio_opts["PYTHON_INCLUDE_DIR"] = python_dir + "/include"

# boost
oiio_opts["BOOST_ROOT"] = excons.GetArgument("with-boost", "", str)

## addtional
oiio_opts["USE_FIELD3D"] = 0
oiio_opts["USE_JPEGTURBO"] = 1
oiio_opts["USE_OPENJPEG"] = 1
oiio_opts["USE_FREETYPE"] = 1
oiio_opts["USE_LIBRAW"] = 1
oiio_opts["USE_OCIO"] = 1
oiio_opts["USE_FFMPEG"] = 0

## extra args
oiio_opts["EXTRA_DSO_LINK_ARGS"] = ""
oiio_opts["EXTRA_CPP_ARGS"] = ""
if sys.platform != "win32":
    oiio_opts["EXTRA_CPP_ARGS"] = " -Wno-deprecated-declarations"


# zlib (no deps [CMake])
def ZlibName(static):
    return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
    return ([] if  static else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "zlib", libnameFunc=ZlibName, definesFunc=ZlibDefines)
if not rv:
    excons.PrintOnce("OIIO: Build zlib from sources ...")
    excons.Call("zlib", imp=["ZlibName", "ZlibPath"])    
    z_static = excons.GetArgument("zlib-static", 1, int)
    z_path = ZlibPath(static=z_static)
    zlib_outputs = [z_path]

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

# bzip2 (no deps [SCons])
def Bzip2Libname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

def Bzip2Defines(static):
    return ([] if static else ["BZ_DLL"])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "bz2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines, varPrefix="BZIP2_")
if not rv:
    excons.PrintOnce("OIIO: Build bzip2 from sources ...")
    excons.Call("bzip2", imp=["BZ2Name", "BZ2Path"])
    bz2_static = excons.GetArgument("bz2-static", 1, int)
    bz2_path = BZ2Path()
    bzip2_outputs = [bz2_path]
    
    oiio_opts["BZIP2_LIBRARY"] = bz2_path
    oiio_opts["BZIP2_INCLUDE_DIR"] = out_incdir

    base = os.path.dirname(os.path.dirname(bz2_path))
    name = BZ2Name()

    freetype_overrides["with-bz2"] = base
    freetype_overrides["bz2-static"] = bz2_static
    freetype_overrides["bz2-name"] = name
else:
    bzip2_outputs = []

oiio_dependecies += bzip2_outputs

# jbig (no deps [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "jbig")
if not rv:
    excons.PrintOnce("OIIO: Build jbig from sources ...")
    excons.Call("jbigkit", imp=["JbigName", "JbigPath"])
    jbig_path = JbigPath()
    jbig_outputs = [jbig_path, out_incdir + "/jbig_ar.h"]

    tiff_overrides["with-jbig"] = os.path.dirname(os.path.dirname(jbig_path))
    tiff_overrides["jbig-static"] = 1
    tiff_overrides["jbig-name"] = JbigName()
else:
    jbig_outputs = []

oiio_dependecies += jbig_outputs
extra_libs.append(JbigPath())

# jpeg (no deps [CMake/Automake])
#jpeg_overrides["libjpeg-jpeg8"] = 1
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libjpeg", varPrefix="JPEG_")
if not rv:
    excons.PrintOnce("OIIO: Build libjpeg-turbo from sources ...")
    excons.Call("libjpeg-turbo", overrides=jpeg_overrides, imp=["LibjpegName", "LibjpegPath"])
    jpeg_static = excons.GetArgument("libjpeg-static", 1, int)
    jpeg_path = LibjpegPath(static=jpeg_static)
    jpeg_outputs = [LibjpegPath(static=jpeg_static)]

    oiio_opts["JPEG_INCLUDE_DIR"] = out_incdir
    oiio_opts["JPEG_LIBRARY"] = jpeg_path

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

# openjpeg (no deps, [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "openjpeg")
if not rv:
    excons.PrintOnce("OIIO: Build openjpeg from sources ...")
    excons.Call("openjpeg", imp=["OpenjpegPath"])
    openjpeg_outputs = [OpenjpegPath()]

    oiio_opts["OPENJPEG_HOME"] = excons.OutputBaseDirectory()
    oiio_opts["OPENJPEG_INCLUDE_DIR"] = out_incdir + "/openjpeg-2.1"
    oiio_opts["OPENJPEG_LIBRARY"] = OpenjpegPath()
    oiio_opts["OPENJPEG_LIBRARY_RELEASE"] = OpenjpegPath()
    oiio_opts["OPENJPEG_LIBRARY_DEBUG"] = OpenjpegPath()
else:
    openjpeg_outputs = []

oiio_dependecies += openjpeg_outputs

# png (depends on zlib, [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libpng", varPrefix="PNG_")
if not rv:
    excons.PrintOnce("OIIO: Build libpng from sources ...")
    excons.cmake.AddConfigureDependencies("libpng", zlib_outputs)
    excons.Call("libpng", overrides=png_overrides, imp=["LibpngName", "LibpngPath"])
    png_static = excons.GetArgument("libpng-static", 1, int)
    png_path = LibpngPath(png_static)
    libpng_outputs = [png_path]

    oiio_opts["PNG_INCLUDE_DIR"] = out_incdir
    oiio_opts["PNG_LIBRARY"] = png_path
    
    base = os.path.dirname(os.path.dirname(png_path))
    name = LibpngName(png_static)
    
    freetype_overrides["with-libpng"] = base
    freetype_overrides["libpng-static"] = png_static
    freetype_overrides["libpng-name"] = name
else:
    libpng_outputs = []

oiio_dependecies += libpng_outputs

# tiff (depends on zlib, jpeg, jbig [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libtiff", varPrefix="TIFF_")
if not rv:
    excons.PrintOnce("OIIO: Build libtiff from sources ...")
    excons.cmake.AddConfigureDependencies("libtiff", jbig_outputs + zlib_outputs + jpeg_outputs)
    excons.Call("libtiff", overrides=tiff_overrides, imp=["LibtiffPath"])
    tiff_path = LibtiffPath()
    tiff_outputs = [tiff_path]

    oiio_opts["TIFF_INCLUDE_DIR"] = out_incdir
    oiio_opts["TIFF_LIBRARY"] = tiff_path
else:
    tiff_outputs = []

oiio_dependecies += tiff_outputs

# lcms2 (depends on jpeg, tiff [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "lcms2", varPrefix="LCMS2_")
if not rv:
    # requires jpeg, tiff (-> jbig, zlib)
    excons.PrintOnce("OIIO: Build lcms from sources ...")
    excons.Call("Little-cms", imp=["LCMS2Name", "LCMS2Path"])
    lcms2_static = excons.GetArgument("lcms2-static", 1, int)
    lcms2_path = LCMS2Path()
    lcms2_outputs = [lcms2_path]

    oiio_opts["LCMS2_LIBRARY"] = lcms2_path
    oiio_opts["LCMS2_INCLUDE_DIR"] = out_incdir

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

oiio_dependecies += lcms2_outputs

# libraw (depends on jpeg, lcms2 [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "libraw")
if not rv:
    excons.PrintOnce("OIIO: Build libraw from sources ...")
    excons.Call("LibRaw", overrides=libraw_overrides, imp=["LibrawPath", "LibrawName"])
    libraw_path = LibrawPath()
    libraw_outputs = [libraw_path]

    oiio_opts["LibRaw_INCLUDE_DIR"] = out_incdir + "/libraw"
    oiio_opts["LibRaw_r_LIBRARIES"] = libraw_path
else:
    libraw_outputs = []

oiio_dependecies += libraw_outputs

# freetype (depends on zlib, bzip2, png [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "freetype")
if not rv:
    excons.PrintOnce("OIIO: Build freetype from sources ...")
    excons.cmake.AddConfigureDependencies("freetype", zlib_outputs + libpng_outputs + bzip2_outputs)
    excons.Call("freetype", overrides=freetype_overrides, imp=["FreetypeName", "FreetypePath"])
    freetype_path = FreetypePath()
    freetype_outputs = [freetype_path]
    
    oiio_opts["FREETYPE_INCLUDE_DIR"] = out_incdir
    oiio_opts["FREETYPE_LIBRARY"] = freetype_path
else:
    freetype_outputs = []

oiio_dependecies += freetype_outputs

# ocio (depends on tinyxml, yaml-cpp, lcms2 [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "OpenColorIO")
if not rv:
    if sys.platform == "win32":
        ocio_overrides["ocio-use-boost"] = 1
    excons.PrintOnce("OIIO: Build OpenColorIO from sources ...")
    excons.Call("OpenColorIO", overrides=ocio_overrides, imp=["OCIOPath", "YamlCppPath", "TinyXmlPath"])
    ocio_static = excons.GetArgument("ocio-static", 1, int) != 0
    ocio_outputs = [OCIOPath(ocio_static), YamlCppPath(), TinyXmlPath()]
    
    oiio_opts["OCIO_INCLUDE_DIR"] = out_incdir
    # oiio_opts["OCIO_LIBRARY"] = OCIOPath(ocio_static)
    oiio_opts["OCIO_LIBRARIES"] = OCIOPath(ocio_static)
    oiio_opts["LCMS2_LIBRARY"] = LCMS2Path()
    oiio_opts["YAML_LIBRARY"] = YamlCppPath()
    oiio_opts["TINYXML_LIBRARY"] = TinyXmlPath()
else:
    ocio_outputs = []

oiio_dependecies += ocio_outputs

# opexnexr (depends on zlib [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "openexr")
if not rv:
    excons.PrintOnce("OIIO: Build openexr from sources ...")
    excons.Call("openexr", overrides=openexr_overrides, imp=["HalfPath", "IexPath", "ImathPath", "IlmThreadPath", "IlmImfPath"])
    openexr_static = (excons.GetArgument("openexr-static", 1, int) != 0)
    openexr_half = HalfPath(openexr_static)
    openexr_iex = IexPath(openexr_static)
    openexr_imath = ImathPath(openexr_static)
    openexr_ilmt = IlmThreadPath(openexr_static)
    openexr_imf = IlmImfPath(openexr_static)
    openexr_outputs = [openexr_half, openexr_iex, openexr_imath, openexr_ilmt, openexr_imf]

    oiio_opts["OPENEXR_HOME"] = out_basedir
    oiio_opts["OPENEXR_INCLUDE_DIR"] = out_incdir
    oiio_opts["OPENEXR_HALF_LIBRARY"] = openexr_half
    oiio_opts["OPENEXR_IEX_LIBRARY"] = openexr_iex
    oiio_opts["OPENEXR_IMATH_LIBRARY"] = openexr_imath
    oiio_opts["OPENEXR_ILMTHREAD_LIBRARY"] = openexr_ilmt
    oiio_opts["OPENEXR_ILMIMF_LIBRARY"] = openexr_imf
else:
    openexr_outputs = []

oiio_dependecies += openexr_outputs


# oiio build
oiio_opts["EXTERNAL_LIBS"] = ";".join(extra_libs)
oiio_opts["EXTERNAL_INCLUDE_DIRS"] = ";".join(extra_includes)

for k, v in oiio_opts.iteritems():
    if isinstance(v, basestring):
        oiio_opts[k] = v.replace("\\", "/")


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
