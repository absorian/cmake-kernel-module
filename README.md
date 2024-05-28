# cmake-kernel-module
CMake template for developing linux kernel modules. Supports clion intellisense. Put a star if helped :)

It works by symlinking the sources into build directory and executing make in it during the build.

## Notes:
- do not name the project (specifically module obj) the same as any of the sources
- do not put sources in folders whose name exists in cmake build template (i.e. CMakeFiles, Testing...)
- same goes for naming sources in project root
