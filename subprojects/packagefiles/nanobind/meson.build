project(
    'nanobind',
    'cpp',
    version: '2.0.0',
    meson_version: '>=0.53.0',
    license: 'BSD-3-Clause',
)

py_mod = import('python')
py = py_mod.find_installation()
py_dep = py.dependency(embed: true)

robin_map_dep = dependency('robin-map')

incdir = include_directories('include')

# Arguments are derived from nanobind_build_library function in CMake configuration
compile_args = []
link_args = []
dep_compile_args = []
dep_link_args = []
if get_option('python.allow_limited_api')
  py_version = py.language_version()

  if py_version.version_compare('>=3.12')
    compile_args += ['-DPy_LIMITED_API=0x030C0000']
  endif
endif

if get_option('buildtype') == 'shared'
  # nanobind_link_options uses linker response files for either PyPy or
  # CPython at this step - how do we support in Meson?
  compile_args += ['-DNB_BUILD']
  dep_compile_args += ['-DNB_SHARED']

  if host_machine.system() == 'darwin'
    link_args += ['-Wl,-dead_strip', '-Wl,x', '-Wl,-S']
  elif host_machine.system() != 'windows'
    link_args += ['-Wl,-s']
  endif
elif host_machine.system() != 'windows' and host_machine.system() != 'darwin'
  dep_compile_args += ['-ffunction-sections', '-fdata-sections']
  dep_link_args += ['-Wl,--gc-sections']
endif

# Here the CMake configuration sets PIC - does Meson handle this internally?

compiler = meson.get_compiler('cpp')
if compiler.get_id() == 'msvc'
  dep_compile_args += ['-D_CRT_SECURE_NO_WARNINGS']
else
  # Discussion in WrapDB PR #1556 as to whether or not this is necessary
  # compile_args += ['-fno-strict-aliasing']
endif

# Here the CMake configuration links against Python::SABIModule on Windows
# if building an abi3 target - does Meson handle this internally?

if get_option('buildtype') == 'release'
  compile_args += [
      '-DNDEBUG',
      '-DNB_COMPACT_ASSERTIONS',
  ]
endif

nanobind_lib = static_library(
    'nanobind',
    sources: [
        'src/nb_internals.cpp',
        'src/nb_func.cpp',
        'src/nb_type.cpp',
        'src/nb_enum.cpp',
        'src/nb_ndarray.cpp',
        'src/nb_static_property.cpp',
        'src/common.cpp',
        'src/error.cpp',
        'src/trampoline.cpp',
        'src/implicit.cpp',
    ],
    include_directories: [incdir],
    dependencies: [py_dep, robin_map_dep],
    cpp_args: compile_args,
    link_args: link_args,
    gnu_symbol_visibility: 'hidden',
)


nanobind_dep = declare_dependency(
    include_directories: [incdir],
    link_with: [nanobind_lib],
    dependencies: [py_dep, robin_map_dep],
    compile_args: dep_compile_args,
    link_args: dep_link_args,
)
