# OpenModelica models in JavaScript

[OpenModelica](http://openmodelica.org) is an open-source compiler for
the [Modelica](http://modelica.org) language. Modelica is a language for
simulating systems (electrical, mechanical, and many more).

The files in this repository include some files to help OpenModelica
compile models to JavaScript using
[Emscripten](http://emscripten.org/).

Thanks to Martin Sjölund (a core OpenModelica developer), OpenModelica
has a backend that supports compilation with Emscripten. 

## Examples

Overall, the user experience works relatively well. All the
calculations are done on the client. Here are several examples:

- [Modelica.Blocks.Examples.PID_Controller](http://tshort.github.io/mdpad/mdpad.html?Modelica.Blocks.Examples.PID_Controller.md)
- [Modelica.Electrical.Analog.Examples.ChuaCircuit](http://tshort.github.io/mdpad/mdpad.html?Modelica.Electrical.Analog.Examples.ChuaCircuit.md)
- [Modelica.Electrical.Analog.Examples.OvervoltageProtection](http://tshort.github.io/mdpad/mdpad.html?Modelica.Electrical.Analog.Examples.OvervoltageProtection.md)
- [Modelica.Electrical.Analog.Examples.Rectifier](http://tshort.github.io/mdpad/mdpad.html?Modelica.Electrical.Analog.Examples.Rectifier.md)
- [Modelica.Thermal.HeatTransfer.Examples.TwoMasses](http://tshort.github.io/mdpad/mdpad.html?Modelica.Thermal.HeatTransfer.Examples.TwoMasses.md)
- [Modelica.Mechanics.MultiBody.Examples.Systems.RobotR3.fullRobot](http://tshort.github.io/mdpad/mdpad.html?Modelica.Mechanics.MultiBody.Examples.Systems.RobotR3.fullRobot.md)

See [here](https://github.com/tshort/mdpad/tree/gh-pages/) for the
source code for these examples.

You wouldn't want to run complex Monte Carlo analysis with this, but
many models should run sufficiently fast for use on the web. With
advances in Emscripten and a few tweaks to OpenModelica, models in
Firefox run within about 1.5X of native, and in the most recent
Chrome, they run within 2X of native. The key to the speed is that
Emscripten compiles to the asm.js format which can be optimized very
well. The first run takes more time because the files need to be
downloaded and compiled. 

The JavaScript file generated by Emscripten is almost 2 MB (about 0.5
MB gzipped) for models with complexity similar to examples in the
Modelica Standard Library. The most complex model I've tried is the
MultiBody.Examples.Loops.EngineV6 that was about 8 MB. Some options to
try to reduce the page size are:

- Look for more stuff to strip out (one could strip out all solvers but
  DASSL for example).
- Replace some of the system code with something already built in. For
  example, it might be possible to replace `expat` with JavaScript's
  own XML processing functionality.

The gui for these pages was done using
[mdpad](http://tshort.github.io/mdpad/), another web technology I'm
experimenting with. You can of course write your own JavaScript gui
that interfaces with the simulation code.

## Setting up Compilation

To set up to compile your own model, here are the steps needed:

- Install [Emscripten](http://emscripten.org/).
- Make sure Emscripten works and paths to `emcc` are set up right.
- Install OpenModelica with at least SVN revision 18828.
- Copy all of the object files (*.so) and pre.js into the build
  directory of OpenModelica at this location (you may need to create
  the directory):
  - build/lib/omc/emcc/

If you want to compile the Modelica libraries (as OpenModelica gets
updated), you can run the following at the shell from your
OpenModelica source location:

    make -j4 -C SimulationRuntime/c emcc
    make -j4 omc

This will update the following files:
  - build/lib/omc/emcc/libf2c.so
  - build/lib/omc/emcc/libSimulationRuntimeC.so

Note that I have only tested this with Linux. I don't know if it will
work under Windows.

## Compiling your model

To compile your own model, create a modelica script (*.mos) file
something like:
    
    loadModel(Modelica);
    setCommandLineOptions("+simCodeTarget=JavaScript");
    buildModel(Modelica.Electrical.Analog.Examples.ChuaCircuit);

Run this in a shell with something like:

    omc chua.mos

This will compile the model to JavaScript. This will generate many
other files, too. The most important files are:

- Modelica.Electrical.Analog.Examples.ChuaCircuit.js
- Modelica.Electrical.Analog.Examples.ChuaCircuit_init.xml
- Modelica.Electrical.Analog.Examples.ChuaCircuit_init.md

The `js` file is the JavaScript code. The `xml` file is the
initialization file. You will likely want to modify some of the
defaults. The `md` file is a Markdown file that describes a user
interface using [mdpad](http://tshort.github.io/mdpad/).

## Set up the Model with a Web Interface

To run your compiled JavaScript model in a browser, clone the mdpad
files from here:

- https://github.com/tshort/mdpad/tree/gh-pages/

Copy the three files (js, xml, and md) created by the call to omc to
your newly cloned mdpad directory.

Launch a web server from within your mdpad directory. I use `python -m
SimpleHTTPServer` which starts a webserver on port 8000.

Browse to your model, something like
`http://localhost:8000/mdpad_local.html?Modelica.Electrical.Analog.Examples.ChuaCircuit_init.md`.

## Fixing up the Model

There are a few tweaks you will normally want to do. First, you can
edit the Markdown file to better describe your model. You can also
change the inputs and outputs in this file. Using the Chua circuit as
an example, the following Markdown code defines the form inputs on the
page. This is YAML code that describes the form elements. In this
example, `L`, `C1`, and `C2` are inputs to the model that are specific
to this simulation, so modify these are add similar entries for your
model.

    ```yaml jquery=dform
    class : form-horizontal
    col1class : col-sm-7
    col2class : col-sm-5
    html: 
      - name: stopTime
        type: number
        bs3caption: Stop time, sec
        value: 10000.0
      - name: intervals
        type: number
        bs3caption: Output intervals
        value: 500
      - name: tolerance
        type: number
        bs3caption: Tolerance
        value: 0.0001
      - name: L
        type: number
        bs3caption: L, henries
        value: 18.0
      - name: C1
        type: number
        bs3caption: C1, farads
        value: 10.0
      - name: C2
        type: number
        bs3caption: C2, farads
        value: 100.0
    ```

Next, you need to tie these input elements to model parameters in the
xml file. Do this by changing a JavaScript block within the Markdown
file. Here is an example for the three variables set in the Chua
example:

    // Set some model parameters
    $xml.find("ScalarVariable[name = 'L.L']").find("Real").attr("start", L)
    $xml.find("ScalarVariable[name = 'C1.C']").find("Real").attr("start", C1)
    $xml.find("ScalarVariable[name = 'C2.C']").find("Real").attr("start", C2)

JavaScript uses the CSV outputFormat, and this can be quite slow in
Emscripten-compiled code. So, simulations run much faster if you
reduce the amount of output from the simulation. You can do that by
changing the `variableFilter` input in the `_init.xml` file. Here is
an example for the Chua circuit to just show two voltages and two
currents:

    variableFilter = "C1.v|C2.v|L.i|Nr.i" />

You could also change this in the `md` file by adding a line like the
following:

    defex = $xml.find("DefaultExperiment")
    defex.attr("variableFilter", "C1.v|C2.v|L.i|Nr.i")

Lastly, this interface shows an image of the model. You need to create
this image file as follows. In OMEdit, open your model. Right click in
the area with your model visible. Select "Export as an image". Save as
the full model name with an `svg` extension in the mdpad directory.
Now, your model should be visible when you reload the model in the
browser.

## Status

Everything here is experimental at this point. I've tried this with
several models. I've left off some system libraries like Sundials, so
some solvers and functionality might not work.

All this code is free and open source. The OpenModelica code is under
various licenses. The files that I made are granted to the public
domain (or alternatively under the MIT license).
