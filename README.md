# scinit
[![CircleCI status](https://img.shields.io/circleci/project/github/uubk/scinit/master.svg?style=shield)](https://circleci.com/gh/uubk/scinit/tree/master)
[![Codecov status](https://img.shields.io/codecov/c/github/uubk/scinit/master.svg?style=popout)](https://codecov.io/gh/uubk/scinit)
![License](https://img.shields.io/github/license/uubk/scinit.svg?style=popout)

A small init daemon for containers. Supervises multiple processes (optionally
restarting them), forwards signals and handles capabilities.

## Why would I need this?
Because in some environments, it might not be desireable to only run a single,
non-root process per container. Some services might need eleveated permissions
(e.g. `CAP_NET_RAW`) and maybe you have another deployment option besides
Kubernetes that you need to support, so splitting every app into multiple
containers is just not possible.

## Building
You'll need CMake, libcap2, Boost::Program_Options, Boost::Signals2,
Boost::Filesysten and yaml-cpp (and libutil and pthreads which should already be
installed). Additionaly, if you build in debug mode (which is the default),
you'll need lcov. On Debian:
`apt install libcap-dev libyaml-cpp-dev libboost-all-dev cmake`. Afterwards,
it's
```
mkdir build
cd build
cmake ..
make -j<nprocs>
```
Other targets:
  * `make format`: If you like, you can additionally install clang, clang-tidy and clang-format to format the code and give you feedback, which is useful if you'd like to contribute patches.
  * `make check` will run the test suite, skipping cases that require root
  * `make cover` will run the test suite, using `lcov` to generate a readable coverage report.

## How to use
You'll need to create a cinit configuration, a sample is shown below.:
```
programs:
  - name: ping
    path: ./ping
    args:
      - -c 4
      - google.ch
    type: oneshot
    uid: 65534
    gid: 65534
    capabilities:
      - CAP_NET_RAW
```
The configuration(YAML) consists of a program element followed by an array of
programs. In the example above, this would start a `./ping -c 4 google.ch` a
single time (`type: oneshot`) as `nobody:nogroup` and grant it `CAP_NET_RAW` so
that it works without root privileges. The only mandatory options from this
example are `name` and `path`. Other options are:

* `user`/`group` These work like uid/gid, except with names. If you specify both numeric options and strings, the strings will take precedence
* `pty` Can be set to `true` to expose a pseudo-TTY to a child process instead of pipes.
* `before`/`after` Can be set to the name of another program argument to indicate that this program needs to be started before or after it. A program is considered started if it has exitted successfully (type oneshot) or is running (type simple). The implementation is faily basic at the moment, e.g. it doesn't actually check for feasibility.
* `env` If set, contains a mixed list of strings and maps. A string names an additional environment variable to be whitelisted, a map names an environment variable to be set to a given value. The value for a variable supports Inja templates, e.g. `{{ vars/USER }}`.

### Invocation
```
src/scinit --help
Options:
  --help                     print this message
  --config arg (=config.yml) path to config file
  --verbose arg (=0)         be verbose
```
`config` can also point to a directory and verbose turns on *a lot of* output.


## Dependencies
This project depends on [gtest+gmock](https://github.com/google/googletest),
[spdlog](https://github.com/gabime/spdlog) and
[inja](https://github.com/pantor/inja), which are pulled in via submodules.
Additionally, CMake's `GoogleTest` module is included for compatibility with
earlier CMake versions. For coverage, we pull in Lars Bilke's
[CodeCoverage](https://github.com/bilke/cmake-modules/blob/add62f1ebc5412e9eb3b846f73770f7ea31440e9/CodeCoverage.cmake) module.
