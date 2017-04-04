import excons
import re


env = excons.MakeBaseEnv()


if not env.CMakeConfigure("oiio", opts=ARGUMENTS):
    sys.exit(1)


cmake_in = env.CMakeInputs(dirs=["src"], patterns=[re.compile(r"^.*\.(cpp)$")])
cmake_out = env.CMakeOutputs()


target = env.CMake(cmake_out, cmake_in)


env.Alias("oiio", target)


env.CMakeClean()
excons.SyncCache()
