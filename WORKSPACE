"""Bazel workspace for the Beancount.

Note that:

- This is using a local Python runtime, which must be 3.10
- The pip package for 'protobuf' version *must* match that which is in use in
  this build.
- Your version of Bazel installed must match that which is used in the github
  actions (see beancount/.bazelversion).

"""
workspace(name="beancount")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "bazel_skylib",
    sha256 = "66ffd9315665bfaafc96b52278f57c7e2dd09f5ede279ea6d39b2be471e7e3aa",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/bazel-skylib/releases/download/1.4.2/bazel-skylib-1.4.2.tar.gz",
        "https://github.com/bazelbuild/bazel-skylib/releases/download/1.4.2/bazel-skylib-1.4.2.tar.gz",
    ],
)
load("@bazel_skylib//:workspace.bzl", "bazel_skylib_workspace")

bazel_skylib_workspace()


http_archive(
    name = "io_bazel_rules_go",
    sha256 = "6dc2da7ab4cf5d7bfc7c949776b1b7c733f05e56edc4bcd9022bb249d2e2a996",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/rules_go/releases/download/v0.39.1/rules_go-v0.39.1.zip",
        "https://github.com/bazelbuild/rules_go/releases/download/v0.39.1/rules_go-v0.39.1.zip",
    ],
)


http_archive(
    name = "bazel_gazelle",
    sha256 = "727f3e4edd96ea20c29e8c2ca9e8d2af724d8c7778e7923a854b2c80952bc405",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/bazel-gazelle/releases/download/v0.30.0/bazel-gazelle-v0.30.0.tar.gz",
        "https://github.com/bazelbuild/bazel-gazelle/releases/download/v0.30.0/bazel-gazelle-v0.30.0.tar.gz",
    ],
)


load("@io_bazel_rules_go//go:deps.bzl", "go_register_toolchains", "go_rules_dependencies")
load("@bazel_gazelle//:deps.bzl", "gazelle_dependencies", "go_repository")

############################################################
# Define your own dependencies here using go_repository.
# Else, dependencies declared by rules_go/gazelle will be used.
# The first declaration of an external repository "wins".
############################################################

go_rules_dependencies()

go_register_toolchains(version = "1.20.5")

gazelle_dependencies()

http_archive(
    name = "rules_python_gazelle_plugin",
    sha256 = "0a8003b044294d7840ac7d9d73eef05d6ceb682d7516781a4ec62eeb34702578",
    strip_prefix = "rules_python-0.24.0/gazelle",
    url = "https://github.com/bazelbuild/rules_python/releases/download/0.24.0/rules_python-0.24.0.tar.gz",
)



rules_python_version = "93f5ea2f01ce7eb870d3ad3943eda5d354cdaac5"

http_archive(
    name = "rules_python",
    strip_prefix = "rules_python-{}".format(rules_python_version),
    url = "https://github.com/bazelbuild/rules_python/archive/{}.zip".format(rules_python_version),
    sha256 = "179541b519e8fd7c8fbfd0d2a2a51835cf7c83bd6a8f0f3fd599a0910d1a0981"
)

load("@rules_python//python:repositories.bzl", "py_repositories", "python_register_toolchains")

py_repositories()

python_register_toolchains(
    name = "python3_10",
    python_version = "3.10",
)

load("@python3_10//:defs.bzl", "interpreter")

load("@rules_python//python:pip.bzl", "pip_parse", "pip_install")

pip_parse(
   name = "pypi",
   requirements_lock = "//requirements:dev_lock.txt",
   python_interpreter_target = interpreter,
)

load("@pypi//:requirements.bzl", "install_deps")

install_deps()

pip_parse(
   name = "pypi_tools",
   requirements_lock = "//requirements:tools_lock.txt",
   python_interpreter_target = interpreter,
)

load("@pypi_tools//:requirements.bzl", install_deps2 = "install_deps")

install_deps2()


#------------------------------------------------------------------------------
# Direct dependencies of Beancount.
#
# Each of the following files defines `http_archive` repositories. We group them
# into various subgroups of related features. This could theoretically be
# serialized to a single file, but we keep them to subdirectories to organize
# all the related files. We use maybe_http_archive() throughout.

# Allow an external python_configure() definition to be pulled from pybind11_bael.
http_archive(
    name = "pybind11_bazel",
    url = "https://github.com/pybind/pybind11_bazel/archive/faf56fb3df11287f26dbc66fdedf60a2fc2c6631.zip",
    strip_prefix = "pybind11_bazel-faf56fb3df11287f26dbc66fdedf60a2fc2c6631",
    sha256 = "a185aa68c93b9f62c80fcb3aadc3c83c763854750dc3f38be1dadcb7be223837",
)  # updated = "2022-11-03",

load("@pybind11_bazel//:python_configure.bzl", "python_configure")
python_configure(name = "local_config_python", python_interpreter_target = interpreter)
bind(
    name = "python_headers",
    actual = "@local_config_python//:python_headers",
)

# Bazel general rules packages
load("//bazel:repositories.bzl", "beancount_dependencies")
beancount_dependencies()

#------------------------------------------------------------------------------
# Indirect dependencies from packages we depend on.
#
# Calling these functions will define more repositories, and if there are
# collisions the ones we depend on above will prevail; hopefully the ones we
# define above are compatible with those required by the packages themselves.

load("@bazel_skylib//:workspace.bzl", "bazel_skylib_workspace")
bazel_skylib_workspace()

load("@rules_foreign_cc//foreign_cc:repositories.bzl", "rules_foreign_cc_dependencies")
rules_foreign_cc_dependencies()

load("@rules_pkg//:deps.bzl", "rules_pkg_dependencies")
rules_pkg_dependencies()

load("@rules_cc//cc:repositories.bzl", "rules_cc_dependencies")
rules_cc_dependencies()

load("@rules_proto//proto:repositories.bzl", "rules_proto_dependencies", "rules_proto_toolchains")
rules_proto_dependencies()
rules_proto_toolchains()

# TODO(blais): Update RE-flex version and code.
# TODO(blais): Upgrade the pybind11 deps.
