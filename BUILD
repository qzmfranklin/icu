licenses(['notice'])
package(default_visibility=['//visibility:__pkg__'])

# How to write this BUILD file and update it?
#
# Download the icu4c source code from a tarball. Extract it. Then go through the
# normal build process:
#       cd source
#       ./configure
#       make -j8 &> build.log
#
# The file, build.log, should contain all information we need to transcribe the
# build of icu4c into Bazel.
#
# High level build instructions:
#       http://userguide.icu-project.org/howtouseicu
#
# The recommended compilation configurations:
#       http://source.icu-project.org/repos/icu/trunk/icu4c/readme.html#RecBuild
#

# TODO: Use config_settings() and select() to build on both Linux and
# Darwin instead of using `uname -r`.

# TODO: A lot of of options are probably not optimal any more.  Should find a
# time to read through the recommended configurations and update to the best
# practice as of that time.

icu_major_version = 60
icu_minor_version = 2

icu_src_dir = '/'.join(['.', PACKAGE_NAME, 'source'])

icu_copts = [
    '-fno-exceptions',
    '-I%s/common' % icu_src_dir,
    '-I%s/i18n' % icu_src_dir,
    '-I%s/io' % icu_src_dir,
    '-I%s/tools/ctestfw' % icu_src_dir,
    '-I%s/tools/toolutil' % icu_src_dir,

    # Suppress known warnings.
    '-Wno-deprecated-declarations',
    '-Wno-sign-compare',
]

icu_linkopts = [
    '-ldl',
    '-lm',
]

native.cc_library(
    name = 'icu-base',
    srcs = native.glob([
        'source/common/**/*.c',
        'source/common/**/*.cpp',
        'source/io/**/*.c',
        'source/io/**/*.cpp',
        'source/i18n/**/*.c',
        'source/i18n/**/*.cpp',
        # Including the layoutex directory leads to the following error:
        #       third_party/icu/layoutex/LXUtilities.cpp:10:28: \
        #           fatal error: layout/LETypes.h: No such file or directory
        #        #include "layout/LETypes.h"
        'layoutex/**/*.c',
        'layoutex/**/*.cpp',

        'source/common/**/*.h',
        'source/io/**/*.h',
        'source/i18n/**/*.h',
        'layoutex/**/*.h',
    ], exclude = [
        'source/common/unicode/*.h',
    ]),
    hdrs = native.glob([
        'source/common/unicode/*.h',
    ]),
    includes = [
        'source/common',
        'source/i18n',
        'source/io',
        'source/tools/ctestfw',
        'source/tools/toolutil',
    ],
    defines = [
        # Starting from ICU 61, the `using namespace icu` is removed by default,
        # rendering the U_USING_ICU_NAMESPACE macro unnecessary.
        'U_USING_ICU_NAMESPACE=0',
        'U_CHARSET_IS_UTF8=1',
        'U_NO_DEFAULT_INCLUDE_UTF_HEADERS=1',

        # TODO: The following macros are recommended by the official
        # install guide to enforce explicit type in constructors.
        # But they break the tests shipped with the source code.
        'UNISTR_FROM_CHAR_EXPLICIT=explicit',
        'UNISTR_FROM_STRING_EXPLICIT=explicit',

        # Without -DU_HAVE_ELF_H=1, the tools/toolutil/pkg_genc.c will not
        # export the symbol 'writeObjectCode'.  Discovered via `VERBOSE=1 make`.
        'U_HAVE_ELF_H=1',
        '_REENRANT=1',
        'U_HAVE_STRTOD_L=1',
    ],
    copts = icu_copts + [
        # required for building the files under common/
        '-DU_COMMON_IMPLEMENTATION',
        # required for building the files under io/
        '-DU_IO_IMPLEMENTATION',
        # required for building the files under i18n/
        '-DU_I18N_IMPLEMENTATION',
    ],
    linkopts = icu_linkopts,
)

native.cc_library(
    name = 'stubdata',
    srcs = [
        'source/stubdata/stubdata.cpp',
    ],
    deps = [
        ':icu-base',
    ],
    copts = icu_copts,
)

native.cc_library(
    name = 'icu-tools',
    srcs = native.glob([
        'source/tools/ctestfw/**/*.c',
        'source/tools/ctestfw/**/*.cpp',
        'source/tools/toolutil/**/*.c',
        'source/tools/toolutil/**/*.cpp',
    ]),
    hdrs = native.glob([
        'source/tools/ctestfw/**/*.h',
        'source/tools/toolutil/**/*.h',
    ]),
    deps = [
        'icu-base',
    ],
    copts = icu_copts + [
        '-DU_TOOLUTIL_IMPLEMENTATION',
    ]
)

# The complete list of tools is:
#
#     derb     genccode   gencmn     gendict    genrb      icupkg     pkgdata
#     genbrk   gencfu     gencnval   gennorm2   gensprep   makeconv   uconv
#
# Only build icupkg and pkgdata here as they are needed for building libicudata.
#
# TODO: There is a bug in bazel that prevents the above tools to be built.  The
# problem is that cc_binary treats :stubdata as a separate library.  As a
# result, at linking time, the object file stubdata.pic.o separated from other
# object files by pairs of `--start-lib` and `--end-lib`, which prevents other
# object files from seeing stubdata.pic.o directly, thus leading to the
# following error:
#       error: undefined reference to 'icudt59_dat'
# There is a workaround to the above described problem.  That is, just build the
# :libicudata target, which depends on :icupkg and :pkgdata.  After `bazel build
# :libicudata`, you can find the `icupkg` and `pkgdata` binaries in
# bazel-bin/third_party/icu/.

native.cc_binary(
    name = 'icupkg',
    srcs = [
        'source/tools/icupkg/icupkg.cpp',
    ],
    deps = [
        ':icu-tools',
        ':stubdata',
    ],
    copts = icu_copts,
    linkopts = icu_linkopts,
)

native.cc_binary(
    name = 'pkgdata',
    srcs = [
        'source/tools/pkgdata/pkgdata.cpp',
        'source/tools/pkgdata/pkgtypes.c',
        'source/tools/pkgdata/pkgtypes.h',
    ],
    deps = [
        ':icu-tools',
        ':stubdata',
    ],
    copts = icu_copts,
    linkopts = icu_linkopts,
)

native.cc_library(
    name = 'icu-data',
    srcs = [
        'libicudata.a',
        'libicudata.so',
    ],
)

native.genrule(
    name = 'icupkg_inc',
    srcs = native.glob([
        'source/runConfigureICU',
        # The files below are used by source/runConfigureICU.
        'source/configure',
        'source/common/unicode/*.h',
        'source/config/*',
        'source/config.sub',
        'source/config.guess',
        'source/install-sh',
        'source/icudefs.mk.in',
        'source/**/Makefile.in',
        'source/**/pkgdataMakefile.in',
    ]),
    outs = [
        'icupkg.inc',
    ],
    cmd = ' && '.join([
        # Detect the ARCH used to configure the icu library.
        'if [ `uname` == "Linux" ]; then ARCH=Linux; elif [ `uname` == "Darwin" ]; then ARCH=MacOSX; fi',
        'CFLAGS="-DU_USING_ICU_NAMESPACE=0 -DU_CHARSET_IS_UTF8=1 -DUNISTR_FROM_CHAR_EXPLICIT=explicit -DUNISTR_FROM_STRING_EXPLICIT=explicit -DU_NO_DEFAULT_INCLUDE_UTF_HEADERS=1" $(location source/runConfigureICU) $$ARCH --with-library-bits=64 &> /dev/null',
        'cd data',
        'make -f pkgdataMakefile &> /dev/null',
        'cd ..',
        'cp data/icupkg.inc $(location icupkg.inc)',
    ]),
)

native.genrule(
    name = 'libicudata',
    srcs = [
        'source/data/in/icudt%dl.dat' % icu_major_version,
        'icupkg.inc',
    ],
    tools = [
        ':icupkg',
        ':pkgdata',
    ],
    outs = [
        'libicudata.a',
        'libicudata.so',
    ],
    cmd = ' && '.join([
        # The 'out' dircectory is a temporary directory.
        'mkdir -p out',
        'mkdir -p out/build',
        'mkdir -p out/build/icudt%dl' % icu_major_version,
        'mkdir -p out/build/icudt%dl/curr' % icu_major_version,
        'mkdir -p out/build/icudt%dl/lang' % icu_major_version,
        'mkdir -p out/build/icudt%dl/region' % icu_major_version,
        'mkdir -p out/build/icudt%dl/zone' % icu_major_version,
        'mkdir -p out/build/icudt%dl/unit' % icu_major_version,
        'mkdir -p out/build/icudt%dl/brkitr' % icu_major_version,
        'mkdir -p out/build/icudt%dl/coll' % icu_major_version,
        'mkdir -p out/build/icudt%dl/rbnf' % icu_major_version,
        'mkdir -p out/build/icudt%dl/translit' % icu_major_version,
        'mkdir -p out/tmp',
        'mkdir -p out/tmp/curr',
        'mkdir -p out/tmp/lang',
        'mkdir -p out/tmp/region',
        'mkdir -p out/tmp/zone',
        'mkdir -p out/tmp/unit',
        'mkdir -p out/tmp/coll',
        'mkdir -p out/tmp/rbnf',
        'mkdir -p out/tmp/translit',
        'mkdir -p out/tmp/brkitr',
        # Invoke icupkg to unpack the icu data.
        '$(location :icupkg) -d out/build/icudt{0}l --list -x \* $(location source/data/in/icudt{0}l.dat) -o out/tmp/icudata.lst'.format(icu_major_version),
        # Invoke pkgdata to package the unpacked icu data into c++ libraries
        # that can be linked against.
        '$(location :pkgdata) -O $(location icupkg.inc) -q -c -s out/build/icudt{0}l -d out -e icudt{0}  -T out/tmp -p icudt{0}l -m static -r {0}.{1} -L icudata out/tmp/icudata.lst &> /dev/null'.format(icu_major_version, icu_minor_version),
        'cp out/libicudata.a $(location libicudata.a)',
        '$(location :pkgdata) -O $(location icupkg.inc) -q -c -s out/build/icudt{0}l -d out -e icudt{0}  -T out/tmp -p icudt{0}l -m dll -r {0}.{1} -L icudata out/tmp/icudata.lst &> /dev/null'.format(icu_major_version, icu_minor_version),
        # In MacOSX use .dylib; In Linux use .so.
        # TODO (zhongming): This is a super hacky hack!
        'if [ `uname` == "Linux" ]; then DYLIB=so; elif [ `uname` == "Darwin" ]; then DYLIB=dylib; fi',
        'cp -H out/libicudata.$$DYLIB $(location libicudata.so)',
        'rm -rf out',
    ]),
)

native.cc_library(
    name = 'icu',
    visibility = ['//visibility:public'],
    deps = [
        ':icu-base',
        ':icu-data',
    ],
    copts = icu_copts,
)
