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
simd = excons.GetArgument("oiio-simd", "0")
verbose = (1 if excons.GetArgument("oiio-verbose", 0, int) != 0 else 0)
out_basedir = excons.OutputBaseDirectory()
out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"
python_dir = excons.GetArgument("oiio-python-dir", "", str)
python_ver = excons.GetArgument("oiio-python-ver", "2.7", str)

prjs = []
oiio_opts = {}
overrides = {}
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
oiio_opts["CODECOV"] = 0
oiio_opts["HIDE_SYMBOLS"] = 1
oiio_opts["USE_SIMD"] = simd
oiio_opts["INSTALL_SYSTEM_RUNTIME"] = 0
oiio_opts["OIIO_BUILD_TESTS"] = 0
oiio_opts["OIIO_BUILD_TOOLS"] = 1
oiio_opts["EMBEDPLUGINS"] = 1
oiio_opts["VERBOSE"] = verbose

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
#   force static link
ARGUMENTS["boost-static"] = "1"
rv = excons.ExternalLibRequire("boost")
if rv["require"]:
   boost_vh = rv["incdir"] + "/boost/version.hpp"
   if not os.path.isfile(boost_vh):
      excons.WarnOnce("No such a file '%s'" % boost_vh, tool="OIIO")
      sys.exit(1)

   e = re.compile(r"define\s+BOOST_VERSION\s+([^\s]+)")
   intver = 0
   strver = ""
   with open(boost_vh, "r") as f:
      for l in f.readlines():
         m = e.search(l)
         if m:
            try:
               intver = int(m.group(1))
               break
            except:
               intver = 0
               pass
   if intver == 0:
      excons.WarnOnce("Cannot figure out Boost version from '%s'" % boost_vh, tool="OIIO")
      sys.exit(1)
   strver = "%d.%d" % (intver / 100000, (intver / 100) % 1000)

   libsuffix = excons.GetArgument("boost-suffix", "")
   if sys.platform == "win32":
      libsuffix += "-vc%d-mt-%s.lib" (int(excons.mscver * 10), strver)
   else:
      libsuffix += ".a"

   libs = []
   for name in ["filesystem", "regex", "system", "thread"]:
      path = rv["libdir"] + "/libboost_" + name + libsuffix
      if not os.path.isfile(path):
         excons.WarnOnce("Invalid Boost %s library '%s'" % (name, path), tool="OIIO")
         sys.exit(1)
      libs.append(path)

   oiio_opts["BOOST_CUSTOM"] = 1
   oiio_opts["Boost_VERSION"] = intver
   oiio_opts["Boost_INCLUDE_DIRS"] = rv["incdir"]
   oiio_opts["Boost_LIBRARY_DIRS"] = rv["libdir"]
   oiio_opts["Boost_LIBRARIES"] = ";".join(libs)

   pylib = rv["libdir"] + "/libboost_python" + libsuffix
   if os.path.isfile(pylib):
      oiio_opts["boost_PYTHON_FOUND"] = 1
      oiio_opts["Boost_PYTHON_LIBRARIES"] = pylib
   else:
      excons.WarnOnce("No valid Boost python found. Skipping python module.", tool="OIIO")
      oiio_opts["boost_PYTHON_FOUND"] = 0   

else:
   excons.WarnOnce("Boost is require to build OpenImageIO, please provide root directory using 'with-boost=' flag", tool="OIIO")
   sys.exit(1)

## addtional
oiio_opts["USE_FIELD3D"] = 0
oiio_opts["USE_JPEGTURBO"] = 1
oiio_opts["USE_OPENJPEG"] = 1
oiio_opts["USE_FREETYPE"] = 1
oiio_opts["USE_LIBRAW"] = 1
oiio_opts["USE_OCIO"] = 1
oiio_opts["USE_FFMPEG"] = 0
oiio_opts["USE_OPENCV"] = 0
oiio_opts["USE_PTEX"] = 0
oiio_opts["USE_GIF"] = 0
oiio_opts["USE_JASPER"] = 0
oiio_opts["USE_NUKE"] = 0
oiio_opts["USE_OPENSSL"] = 0
# Not building 'iv' so far
oiio_opts["USE_QT"] = 0
oiio_opts["USE_OPENGL"] = 0
oiio_opts["FORCE_OPENGL_1"] = 0

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
if not rv["require"]:
    excons.PrintOnce("OIIO: Build zlib from sources ...")
    excons.Call("zlib", imp=["ZlibName", "ZlibPath"])    
    z_static = excons.GetArgument("zlib-static", 1, int)
    z_path = ZlibPath(static=z_static)
    zlib_outputs = [z_path]

    oiio_opts["ZLIB_INCLUDE_DIR"] = out_incdir
    oiio_opts["ZLIB_LIBRARY"] = z_path

    overrides["with-zlib"] = os.path.dirname(os.path.dirname(z_path))
    overrides["zlib-static"] = z_static
    overrides["zlib-name"] = ZlibName(static=z_static)
else:
    zlib_outputs = []

# bzip2 (no deps [SCons])
def Bzip2Libname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

def Bzip2Defines(static):
    return ([] if static else ["BZ_DLL"])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "bz2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines, varPrefix="BZIP2_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build bzip2 from sources ...")
    excons.Call("bzip2", imp=["BZ2Name", "BZ2Path"])
    bz2_static = excons.GetArgument("bz2-static", 1, int)
    bz2_path = BZ2Path()
    bzip2_outputs = [bz2_path]
    
    oiio_opts["BZIP2_LIBRARY"] = bz2_path
    oiio_opts["BZIP2_INCLUDE_DIR"] = out_incdir

    overrides["with-bz2"] = os.path.dirname(os.path.dirname(bz2_path))
    overrides["bz2-static"] = bz2_static
    overrides["bz2-name"] = BZ2Name()
else:
    bzip2_outputs = []

oiio_dependecies += bzip2_outputs

# jbig (no deps [SCons])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "jbig")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build jbig from sources ...")
    excons.Call("jbigkit", imp=["JbigName", "JbigPath"])
    jbig_path = JbigPath()
    jbig_outputs = [jbig_path, out_incdir + "/jbig_ar.h"]

    overrides["with-jbig"] = os.path.dirname(os.path.dirname(jbig_path))
    overrides["jbig-static"] = 1
    overrides["jbig-name"] = JbigName()

    extra_libs.append(JbigPath())
else:
    jbig_outputs = []
    if not rv["libpath"]:
        excons.WarnOnce("Could not find JBIG library", tool="OIIO")
        sys.exit(1)
    extra_libs.append(rv["libpath"])

oiio_dependecies += jbig_outputs

# jpeg (no deps [CMake/Automake])
#overrides["libjpeg-jpeg8"] = 1
def JpegLibname(static):
    return "jpeg"

rv = excons.cmake.ExternalLibRequire(oiio_opts, "libjpeg", libnameFunc=JpegLibname, varPrefix="JPEG_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build libjpeg-turbo from sources ...")
    excons.Call("libjpeg-turbo", overrides=overrides, imp=["LibjpegName", "LibjpegPath"])
    jpeg_static = excons.GetArgument("libjpeg-static", 1, int)
    jpeg_path = LibjpegPath(static=jpeg_static)
    jpeg_outputs = [LibjpegPath(static=jpeg_static)]

    oiio_opts["JPEG_INCLUDE_DIR"] = out_incdir
    oiio_opts["JPEG_LIBRARY"] = jpeg_path

    overrides["with-libjpeg"] = os.path.dirname(os.path.dirname(jpeg_path))
    overrides["libjpeg-static"] = jpeg_static
    overrides["libjpeg-name"] = LibjpegName(jpeg_static)
else:
    jpeg_outputs = []

oiio_dependecies += jpeg_outputs

# openjpeg (no deps [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "openjpeg")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build openjpeg from sources ...")
    excons.Call("openjpeg", imp=["OpenjpegPath"])
    openjpeg_outputs = [OpenjpegPath()]

    oiio_opts["OPENJPEG_HOME"] = excons.OutputBaseDirectory()
    oiio_opts["OPENJPEG_INCLUDE_DIR"] = out_incdir + "/openjpeg-2.1"
    oiio_opts["OPENJPEG_LIBRARY"] = OpenjpegPath()
    oiio_opts["OPENJPEG_LIBRARY_RELEASE"] = OpenjpegPath()
    oiio_opts["OPENJPEG_LIBRARY_DEBUG"] = OpenjpegPath()
else:
    oiio_opts["OPENJPEG_HOME"] = os.path.dirname(rv["incdir"])
    oiio_opts["OPENJPEG_INCLUDE_DIR"] = rv["incdir"] + "/openjpeg-2.1"
    openjpeg_outputs = []

oiio_dependecies += openjpeg_outputs

# png (depends on zlib [CMake])

def PngLibname(static):
    return "png"

def PngDefines(static):
    return (["PNG_USE_DLL"] if (not static and sys.platform == "win32") else [])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "libpng", libnameFunc=PngLibname, definesFunc=PngDefines, varPrefix="PNG_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build libpng from sources ...")
    excons.cmake.AddConfigureDependencies("libpng", zlib_outputs)
    excons.Call("libpng", overrides=overrides, imp=["LibpngName", "LibpngPath"])
    png_static = excons.GetArgument("libpng-static", 1, int)
    png_path = LibpngPath(png_static)
    libpng_outputs = [png_path]

    oiio_opts["PNG_INCLUDE_DIR"] = out_incdir
    oiio_opts["PNG_LIBRARY"] = png_path
    
    overrides["with-libpng"] = os.path.dirname(os.path.dirname(png_path))
    overrides["libpng-static"] = png_static
    overrides["libpng-name"] = LibpngName(png_static)
else:
    libpng_outputs = []

oiio_dependecies += libpng_outputs

# tiff (depends on zlib, jpeg, jbig [CMake])

def TiffLibname(static):
    return "tiff"

rv = excons.cmake.ExternalLibRequire(oiio_opts, "libtiff", libnameFunc=TiffLibname, varPrefix="TIFF_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build libtiff from sources ...")
    excons.cmake.AddConfigureDependencies("libtiff", jbig_outputs + zlib_outputs + jpeg_outputs)
    excons.Call("libtiff", overrides=overrides, imp=["LibtiffName", "LibtiffPath"])
    tiff_path = LibtiffPath()
    tiff_outputs = [tiff_path]

    oiio_opts["TIFF_INCLUDE_DIR"] = out_incdir
    oiio_opts["TIFF_LIBRARY"] = tiff_path

    overrides["with-libtiff"] = os.path.dirname(os.path.dirname(tiff_path))
    overrides["libtiff-static"] = excons.GetArgument("libtiff-static", 1, int)
    overrides["libtiff-name"] = LibtiffName()
else:
    tiff_outputs = []

oiio_dependecies += tiff_outputs

# lcms2 (depends on jpeg, tiff [SCons])

def Lcms2Defines(static):
    return (["CMS_DLL"] if not static else [])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "lcms2", definesFunc=Lcms2Defines, varPrefix="LCMS2_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build lcms from sources ...")
    excons.Call("Little-cms", overrides=overrides, imp=["LCMS2Name", "LCMS2Path"])
    lcms2_static = excons.GetArgument("lcms2-static", 1, int)
    lcms2_path = LCMS2Path()
    lcms2_outputs = [lcms2_path]

    oiio_opts["LCMS2_LIBRARY"] = lcms2_path
    oiio_opts["LCMS2_INCLUDE_DIR"] = out_incdir

    overrides["with-lcms2"] = os.path.dirname(os.path.dirname(lcms2_path))
    overrides["lcms2-static"] = lcms2_static
    overrides["lcms2-name"] = LCMS2Name()
    overrides["libraw-with-lcms2"] = 1
else:
    lcms2_outputs = []

oiio_dependecies += lcms2_outputs

# libraw (depends on jpeg, lcms2 [SCons])

def RawLibname(static):
    return ("raw" if sys.platform != "win32" else "libraw")

def RawDefines(static):
    return (["LIBRAW_NODLL"] if static else [])

rv = excons.cmake.ExternalLibRequire(oiio_opts, "libraw", libnameFunc=RawLibname, definesFunc=RawDefines, varPrefix="LibRaw_")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build libraw from sources ...")
    excons.Call("LibRaw", overrides=overrides, imp=["LibrawPath", "LibrawName"])
    libraw_path = LibrawPath()
    libraw_outputs = [libraw_path]

    oiio_opts["LibRaw_INCLUDE_DIR"] = out_incdir + "/libraw"
    oiio_opts["LibRaw_r_LIBRARIES"] = libraw_path
else:
    libraw_outputs = []
    oiio_opts["LibRaw_INCLUDE_DIR"] = rv["incdir"] + "/libraw"
    oiio_opts["LibRaw_r_LIBRARIES"] = rv["libpath"]

oiio_dependecies += libraw_outputs

# freetype (depends on zlib, bzip2, png [CMake])
rv = excons.cmake.ExternalLibRequire(oiio_opts, "freetype")
if not rv["require"]:
    excons.PrintOnce("OIIO: Build freetype from sources ...")
    excons.cmake.AddConfigureDependencies("freetype", zlib_outputs + libpng_outputs + bzip2_outputs)
    excons.Call("freetype", overrides=overrides, imp=["FreetypeName", "FreetypePath"])
    freetype_path = FreetypePath()
    freetype_outputs = [freetype_path]
    
    oiio_opts["FREETYPE_INCLUDE_DIR"] = out_incdir
    oiio_opts["FREETYPE_LIBRARY"] = freetype_path
else:
    freetype_outputs = []

oiio_dependecies += freetype_outputs

# ocio (depends on tinyxml, yaml-cpp, lcms2 [SCons])

def OCIOLibname(static):
    return "OpenColorIO"

rv = excons.cmake.ExternalLibRequire(oiio_opts, "ocio", libnameFunc=OCIOLibname)
if not rv["require"]:
    if sys.platform == "win32":
        overrides["ocio-use-boost"] = 1
    excons.PrintOnce("OIIO: Build OpenColorIO from sources ...")
    excons.Call("OpenColorIO", overrides=overrides, imp=["OCIOPath", "YamlCppPath", "TinyXmlPath"])
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
if not rv["require"]:
    excons.PrintOnce("OIIO: Build openexr from sources ...")
    excons.Call("openexr", overrides=overrides, imp=["HalfPath", "IexPath", "ImathPath", "IlmThreadPath", "IlmImfPath"])
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

    oiio_opts["OPENEXR_HOME"] = os.path.dirname(rv["incdir"])
    oiio_opts["OPENEXR_INCLUDE_DIR"] = rv["incdir"]
    oiio_opts["OPENEXR_HALF_LIBRARY"] = rv["libdir"] + "/" + os.path.basename(rv["libpath"]).replace("openexr", "Half")
    oiio_opts["OPENEXR_IEX_LIBRARY"] = rv["libdir"] + "/" + os.path.basename(rv["libpath"]).replace("openexr", "Iex")
    oiio_opts["OPENEXR_IMATH_LIBRARY"] = rv["libdir"] + "/" + os.path.basename(rv["libpath"]).replace("openexr", "Imath")
    oiio_opts["OPENEXR_ILMTHREAD_LIBRARY"] = rv["libdir"] + "/" + os.path.basename(rv["libpath"]).replace("openexr", "IlmThread")
    oiio_opts["OPENEXR_ILMIMF_LIBRARY"] = rv["libdir"] + "/" + os.path.basename(rv["libpath"]).replace("openexr", "IlmImf")

oiio_dependecies += openexr_outputs


# oiio build
oiio_opts["EXTERNAL_LIBS"] = ";".join(extra_libs)
oiio_opts["EXTERNAL_INCLUDE_DIRS"] = ";".join(extra_includes)

for k, v in oiio_opts.iteritems():
    if isinstance(v, basestring):
        oiio_opts[k] = v.replace("\\", "/")

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


prjs.append({"name": "oiio",
             "type": "cmake",
             "cmake-opts": oiio_opts,
             "cmake-cfgs": excons.CollectFiles(["src"], patterns=["CMakeLists.txt"], recursive=True) + ["CMakeLists.txt"] + oiio_dependecies,
             "cmake-srcs": excons.CollectFiles(["src"], patterns=["*.cpp"], recursive=True),
             "cmake-outputs": map(lambda x: out_incdir + "/OpenImageIO/" + os.path.basename(x), excons.glob("src/include/OpenImageIO/*.h")) +
                              [OiioPath(staticlib)]})

excons.AddHelpOptions(oiio="""OPENIMAGEIO OPTIONS
  oiio-static=0|1        : Toggle between static and shared library build [0]
  oiio-simd=0|<simd>     : Use SIMD directives [0]
                           <simd> should be on of sse2, sse3, ssse3, sse4.1, sse4.2, avx, avx2, avx512f, f16c
  oiio-verbose=0|1       : Print lots of messages while compiling [0]
  oiio-python-dir=<path> : Python prefix ['']
  oiio-python-ver=<ver>  : Python version ['2.7']""")

excons.DeclareTargets(env, prjs)

Default("oiio")

Export("OiioName OiioPath RequireOiio")

# Ecosystem

if "eco" in COMMAND_LINE_TARGETS:
    ecoroot = "/" + excons.EcosystemPlatform()
    outdir = excons.OutputBaseDirectory()
    binext = (".exe" if sys.platform == "win32" else "")

    tgts = {"oiio-lib": Glob(outdir + "/lib/*OpenImageIO*"),
            "oiio-inc": Glob(outdir + "/include/OpenImageIO/*"),
            "oiio-bin": map(lambda x: File(outdir + "/bin/" + x + binext), ["iconvert", "idiff", "igrep", "iinfo", "maketx", "oiiotool"]) +
                        ([] if sys.platform != "win32" else Glob(outdir + "/bin/*OpenImageIO*.dll")),
            "oiio-python": Glob(outdir + ("/lib/python/site-packages/*" if sys.platform != "win32" else "/python"))}

    tgtdirs = {"oiio-lib": ecoroot + "/lib",
               "oiio-inc": ecoroot + "/include/OpenImageIO",
               "oiio-bin": ecoroot + "/bin",
               "oiio-python": ecoroot + "/lib/python/%s" % python_ver}

    excons.EcosystemDist(env, "oiio.env", tgtdirs, targets=tgts, name="OpenImageIO", version="%d.%d.%d" % (major, minor, patch))
