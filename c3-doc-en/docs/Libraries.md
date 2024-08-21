Note, the _library system_ is in _early alpha_, _everything below is subject to change_

__C3__ allows convenient packaging of __C3__ source files optionally with statically or dynamically linked libraries.
To use such a _library_, simply pass the path to the _library_ directory and add the library you wish to link to.
The compiler will resolve any dependencies to other _libraries_ and only compile those that are in use.

# How it works
A library may be used either _packaged_ or _unpacked_.
If unpacked, it is simply a directory with the `.c3l` __suffix__, which contains all the necessary files, if packed, this is simply a compressed variant of a directory with the same structure.

## The specification
In the top of the _library_ resides the `manifest.json` file which has the following __structure__:

```json
{
  "provides" : "my_lib",
  "execs" : [],
  "targets" : {
    "macos-x64" : {
      "linkflags" : [],
      "dependencies" : [],
      "linked-libs" : ["my_lib_static", "Cocoa.framework", "c"]
    },
    "windows-x64" : {
      "linkflags" : ["/stack:65536"],
      "dependencies" : ["ms_my_extra"],
      "linked-libs" : ["my_lib_static", "kernel32"],
      "execs" : [],
    }
  }
}
```

In the example here, this library supports two targets: __macos-x64__ and __windows-x64__.
If we tried to use it with any other target, the _compiler_ would give an error.

We see that if we use the `windows-x64` target it will also load the __ms_my_extra__ library.
And we also see that the _linker_ would have a special argument on that platform.

Both targets expect `my_lib_static` to be available for linking.
If this library provides this or _dynamic library_ it will be in the target __sub-directories__, so it likely has the path `windows-x64/my_lib_static.lib` and `macos-z64/libmy_lib_static.a`.

# Source code
Aside from the manifest, __C3__ will read any __C__ and __C3__ files in the same directory as `manifest.json` as well as any files in the target _subdirectory_ for the current target.
For __static libraries__ typically a `.c3i` file (_that is, a __C3__ file without any implementations_) is provided, similar to how __.h__ files are used in __C__.

## Additional actions

`"exec"`, which is available both at the top level and _per-target_, lists the scripts which will be invoked when a library is used.
This requires running the compiler at full trust level using the `--trust=full` option.

## How to – automatically – export libraries
_This is not implemented yet, docs will materialize once it is finished_