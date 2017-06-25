# To build the icu library inside another Bazel project, put the following
# snippet into the WORKSPACE file:
#       ```
#       git_repository(
#           name = 'com_github_qzmfranklin_icu',
#           commit = '<COMMIT_SHA_HASH_HERE>',
#           remote = 'https://github.com/qzmfranklin/icu.git',
#       )
#
#       bind(
#           name = 'icu',
#           actual = '@com_github_qzmfranklin_icu//:icu',
#       )
#       ```

licenses(['notice'])
load(':icu.bzl', 'icu_library')
icu_library()
