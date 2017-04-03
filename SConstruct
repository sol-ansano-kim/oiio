import excons
import re


oiio_opts ={}


env = excons.MakeBaseEnv()


if not env.CMakeConfigure("oiio", opts=oiio_opts):
    sys.exit(1)


cmake_in = env.CMakeInputs(dirs=["."], patterns=[re.compile(r"^.*\.(h|c|S)$")])
cmake_out = env.CMakeOutputs()


target = env.CMake(cmake_out, cmake_in)


env.CMakeClean()
env.Alias("oiio", target)


excons.SyncCache()
