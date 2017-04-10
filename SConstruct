import excons
import re
import sys
import excons


env = excons.MakeBaseEnv()
staticlib = (excons.GetArgument("oiio-static", 0, int) != 0)


prjs = [{"name": "oiio",
         "type": "cmake",
         "cmake-opts": {"BUILDSTATIC": staticlib},
         "cmake-cfgs": excons.CollectFiles(["."], patterns=["CMakeLists.txt"], recursive=True),
         "cmake-srcs": excons.CollectFiles(["src"], patterns=["*.cpp"], recursive=True)}]


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


Export("OiioName OiioPath RequireOiio")
