fips-glslang
=========

[![Build Status](https://travis-ci.org/fungos/fips-glslang.svg?branch=travis-ci)](https://travis-ci.org/fungos/fips-glslang)

fipsified glslang (https://github.com/KhronosGroup/fips-glslang)

fips build system: https://github.com/floooh/fips

To use glslang you need just to add it to your `fips.yml`:

```cmake
imports:
     fips-glslang:
         git: https://github.com/fungos/fips-glslang.git
```

And them add a dependency to your project using `fips_deps(OGLCompiler SPIRV hlsl glslang OSDependent)`.