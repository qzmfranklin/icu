licenses(['notice'])
package(default_visibility = ['//visibility:__pkg__'])

icu_copts = [
    '-fno-exceptions',
    '-Isource/common',
    '-Isource/i18n',
    '-Isource/io',
    '-Isource/tools/ctestfw',
    '-Isource/tools/toolutil',
]

icu_linkopts = [
    '-ldl',
]

cc_library(
    visibility = ['//visibility:public'],
    name = 'icu',
    deps = [
        ':icu-base',
        ':icu-data',
    ],
    copts = icu_copts,
)

cc_library(
    name = 'icu-base',
    srcs = glob([
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
        #'layoutex/**/*.c',
        #'layoutex/**/*.cpp',
    ]),
    hdrs = glob([
        'source/common/**/*.h',
        'source/io/**/*.h',
        'source/i18n/**/*.h',
        #'layoutex/**/*.h',
    ]),
    includes = [
        # Use this 'includes' directive to
        'source/common',
        'source/i18n',
        'source/io',
        'source/tools/ctestfw',
        'source/tools/toolutil',
    ],
    defines = [
        # Officially recommended settings.
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

cc_library(
    name = 'stubdata',
    srcs = [
        'source/stubdata/stubdata.cpp',
    ],
    deps = [
        ':icu-base',
    ],
    copts = icu_copts,
)

cc_library(
    name = 'icu-tools',
    srcs = glob([
        'source/tools/ctestfw/**/*.c',
        'source/tools/ctestfw/**/*.cpp',
        'source/tools/toolutil/**/*.c',
        'source/tools/toolutil/**/*.cpp',
    ]),
    hdrs = glob([
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

cc_binary(
    name = 'icupkg',
    srcs = [
        'source/tools/icupkg/icupkg.cpp',
    ],
    deps = [
        ':icu-tools',
        ':stubdata',
    ],
    copts = icu_copts,
)

cc_binary(
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
)

genrule(
    name = 'icupkg_inc',
    srcs = glob([
        'source/runConfigureICU',
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
    cmd = 'true' +
        ' && CFLAGS="-DU_USING_ICU_NAMESPACE=0 -DU_CHARSET_IS_UTF8=1 -DUNISTR_FROM_CHAR_EXPLICIT=explicit -DUNISTR_FROM_STRING_EXPLICIT=explicit -DU_NO_DEFAULT_INCLUDE_UTF_HEADERS=1" ./source/runConfigureICU Linux --with-library-bits=64 &> /dev/null' +
        ' && cd data' +
        ' && make -f pkgdataMakefile &> /dev/null' +
        ' && cd ..' +
        ' && cp data/icupkg.inc $(location icupkg.inc)' +
        '',
)

genrule(
    name = 'libicudata',
    srcs = [
        'source/data/in/icudt59l.dat',
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
    cmd = 'true' +
        # The 'out' dircectory is a temporary directory.
        ' && mkdir -p out' +
        ' && mkdir -p out/build' +
        ' && mkdir -p out/build/icudt59l' +
        ' && mkdir -p out/build/icudt59l/curr' +
        ' && mkdir -p out/build/icudt59l/lang' +
        ' && mkdir -p out/build/icudt59l/region' +
        ' && mkdir -p out/build/icudt59l/zone' +
        ' && mkdir -p out/build/icudt59l/unit' +
        ' && mkdir -p out/build/icudt59l/brkitr' +
        ' && mkdir -p out/build/icudt59l/coll' +
        ' && mkdir -p out/build/icudt59l/rbnf' +
        ' && mkdir -p out/build/icudt59l/translit' +
        ' && mkdir -p out/tmp' +
        ' && mkdir -p out/tmp/curr' +
        ' && mkdir -p out/tmp/lang' +
        ' && mkdir -p out/tmp/region' +
        ' && mkdir -p out/tmp/zone' +
        ' && mkdir -p out/tmp/unit' +
        ' && mkdir -p out/tmp/coll' +
        ' && mkdir -p out/tmp/rbnf' +
        ' && mkdir -p out/tmp/translit' +
        ' && mkdir -p out/tmp/brkitr' +
        # Invoke icupkg to unpack the icu data.
        ' && $(location :icupkg) -d out/build/icudt59l --list -x \* $(location source/data/in/icudt59l.dat) -o out/tmp/icudata.lst' +
        # Invoke pkgdata to package the unpacked icu data into c++ libraries
        # that can be linked against.
        ' && $(location :pkgdata) -O $(location icupkg.inc) -q -c -s out/build/icudt59l -d out -e icudt59  -T out/tmp -p icudt59l -m static -r 59.1 -L icudata out/tmp/icudata.lst &> /dev/null' +
        ' && cp out/libicudata.a $(location libicudata.a)' +
        ' && $(location :pkgdata) -O $(location icupkg.inc) -q -c -s out/build/icudt59l -d out -e icudt59  -T out/tmp -p icudt59l -m dll -r 59.1 -L icudata out/tmp/icudata.lst &> /dev/null' +
        ' && cp -H out/libicudata.so $(location libicudata.so)' +
        ' && rm -rf out' +
        '',
)

cc_library(
    name = 'icu-data',
    srcs = [
        'libicudata.a',
        'libicudata.so',
    ],
)
