# Converting OpenModelica models to JavaScript

[OpenModelica](http://openmodelica.org) is an open-source compiler for
the [Modelica](http://modelica.org) language. Modelica is language for
simulating systems (electrical, mechanical, and many more).

The files in this repository include the C source generated by
OpenModelica as well as the Makefile set up to compile to JavaScript
using [Emscripten](http://emscripten.org/).

See [here](http://tshort.github.io/mdpad/mdpad.html?chua.md) for a
webpage that runs the `ChuaCircuit` model from
Modelica.Electrical.Analog.Examples. The gui for that page was done
using [mdpad](http://tshort.github.io/mdpad/), another web technology
I'm experimenting with.

Overall, the user experience works relatively well. Granted, the Chua
circuit is a simple model, but the simulations execute reasonably
fast. You wouldn't want to run Monte Carlo analysis, but many models
should run sufficiently fast for use on the web. The biggest issue is
the initial page loading is slow. The JavaScript file generated by
Emscripten is almost 2 MB (about 0.5 MB gzipped). Some options to try
to reduce the page size are:

- Look for more stuff to strip out (one could strip out all solvers but
  DASSL for example).
- Replace some of the system code with something already built in. For
  example, it might be possible to replace `expat` with JavaScript's
  own XML processing functionality.
- Run this through another optimizer like
  [UglifyJS](http://lisperator.net/uglifyjs/).
- Try out other Emscripten optimizing options.

## Compiling your model

To compile your own model, here are the steps needed:

- Install [Emscripten](http://emscripten.org/).
- Make sure Emscripten works and paths to `emcc` are set up right.
- Install `f2c`.
- Copy the header file `/usr/include/f2c.h` to
  `~/emscripten/system/include` (or wherever your emscripten is installed).
- Adjust the filenames and paths in the `Makefile` as appropriate.
- Run `emmake make`.

This should give you the file `main_mod.js`. You can try running it
with `node main_mod.js`. It will fail because the xml initiation file
hasn't been loaded. You can modify the `Makefile` to load this file
automatically, but when writing a gui, I found that it was easier to
load the xml file in JavaScript, modify it, then write it to the
Emscripten file system.

## Making a GUI that uses the JavaScript model

To use `main_mod.js` on your webpage, here are the steps you need. 

- Comment out the `run()` statement near the end of `main_mod.js`.
  `run()` runs the model, but we don't want this to happen when the
  page loads (the xml init file isn't ready).
- Load `main_mod.js` into your webpage.
- Load the `whatever_init.xml` into JavaScript, modify it as needed
  then write it to the Emscripten file system.
- To run the model, execute `run()` in JavaScript. 
- Read the results from the Emscripten file system, and plot or do
  other manipulations. You probably want to set `outputFormat` to
  `"csv"` for easier processing.

For an example gui (also open source), see
[here](http://tshort.github.io/mdpad/mdpad.html?chua.md). The Markdown
source for this example is
[here](http://tshort.github.io/mdpad/chua.md). 

## Compiling the OpenModelica libraries

This repository includes the `libSimulationRuntimeC.so`, `libf2c.a`, and
`libexpat.so` needed to compile models. If you want to compile these
yourself, then in addition to the steps above, you need to do the
following:

- In `openmodelica/SimulationRuntime/c`, run `emmake make -f
  Makefile.emscripten`. 
- The only change I remember making to the C runtime was in the file
  `openmodelica/SimulationRuntime/c/simulation/simulation_runtime.cpp`
  where I added a return to avoid the `regcomp` routine which I guess
  that Emscripten doesn't support. Given that, the `filter`
  functionality of simulations won't work.
- Compile `f2c` using `emcc` in
  `openmodelica/SimulationRuntime/c/simulation/libf2c`. I don't think
  I made any changes, just `emmake make -f Makefile.emscripten`.
- Compile `expat` using `emcc`. I don't think I made any changes,
  just `emconfigure configure` then `emmake make`. 

The `expat` and `SimulationRuntime` code is included in this
repository. These subdirectories are not needed to compile code, but
they may be useful for recompiling the libraries with Emscripten. Note
that the `om_SimulationRuntime_c` subdirectory needs to be in the
`openmodelica/SimulationRuntime/c` directory structure.

## Status

Everything here is extremely experimental at this point. I've only
tried this one model. I've left off some system libraries like
Sundials, LAPACK, and others, so some functionality might not work.

All this code is free and open source. The OpenModelica code is under
various licenses. The Makefiles and other configuration files that I
made are granted to the public domain (or alternatively under the MIT
license). 
