Building a project is done by invoking the __C3__ compiler with the `build` or `run` command inside of the project structure.
The compiler will search upwards in the file hierarchy until a _`project.json`_ file is found.

## Project Structure
# Customizing A Project
A new project is provided with a barebone structure in `project.json`:
```json
{
  "version": "0.1.0",
  "authors": [
    "John Doe <john.doe@example.com>"
  ],
  " ": "1",
  "warnings": [ "no-unused" ],
  // sources compiled
  "sources": [ "./**" ],
  // directories where C3 library files may be found
  "dependency-search-paths": [ "lib" ],
  // libraries to use
  "dependencies": [],
  // c compiler
  "cc": "cc",
  // c sources
  "c-sources": [ "./c-source/**" ],
  "targets": {
    "hello_world": {
      "type": "executable"
    }
  }
}
```

By default, an executable in assumed, but changing the type to `"static-lib"` or `"dynamic-lib"` creates static library and dynamic library targets respectively.

_This part will be updated, stay tuned_

# Compilation options
The project file contains common settings at the top level, that can be overridden by each target, by simply assigning that particular key.
So if the top level defines `target` to be `macos-x64` and the actual target defines it to be `windows-x64`, then the `windows-x64` will be used for compilation.

Similarly, compiler command line parameters can be used in turn to override the target setting.

## targets
The list of targets that can be built.

## dependencies
List of __C3__ libraries (_“.c3l”_) to use when compiling the target.

## sources
List of source files to compile.

## cc
__C__ compiler to use for compiling __C__ sources (if __C__ sources are compiled together with __C3__ files).

## c-sources
List of __C__ sources to compile.

## version
_Not handled yet_

Version for library, will also be provided as a compile time constant.

## authors
_Not handled yet_

List of authors to add for library compilation.

## langrev
_Not handled yet_

The language revision to use.

## config
_Not added yet_

Under the config you define external constants (_“key: value”_) that will be included in compilation as if they were global macro constants.

## export
_Not added yet_

Define the list of modules to be exported by a library. Not valid for executables.

## warnings
_Not completely supported yet_

List of warnings to enable during compilation.

# Target options
## type
This mandatory option should be one of _“executable”_, _“dynamic-lib”_ and _“static-lib”_.

More types will be added

# Using environment variables
_Not supported yet_

In addition to constants any values starting with __”$”__ will be assumed to be environment variables.

For example _“$HOME”_ would on unix systems return the home directory.
For strings that start with _$_ but should not be interpreted as an environment variable.
For example, the string "_\$HOME_" would be interpreted as the plain string “_$HOME_”
