FastOutput class
================

Provides a convenient interface to save high-time-resolution output at a few points from a
BOUT++ simulation. Gets run-time options by default from the ``[fast_output]`` section.
Can produce output every internal timestep (fastest possible rate, but not regular), by
setting ``type = timestep`` or at a fixed multiple of the output timestep, by setting
``type = monitor``. [Warning: if the resulting monitor timestep is not long enough to
contain many internal timesteps then performance may be significantly impacted.] To use,
add a ``FastOutput`` member to your ``PhysicsModel``. Also need to add fields and a
position given by global indices to the object. For example, to read an arbitrary number
of points as fractions of the box size from options, and add density and temperature at
those points to ``FastOutput fast_output``, you could add the following to your ``init()``
method.

    // Set up output for synthetic Langmuir probe trace
    if (fast_output.enabled) {
  
      // Add monitor if necessary
      if (fast_output.enable_monitor) {
        solver->addMonitor(&fast_output);
      }
  
      // Add points from the input file
      int i = 0;
      BoutReal xpos, ypos, zpos;
      int ix, iy, iz;
      Options* fast_output_options = Options::getRoot()->getSection("fast_output");
      while (true) {
        // Add more points if explicitly set in input file
        fast_output_options->get("xpos"+std::to_string(i), xpos, -1.);
        fast_output_options->get("ypos"+std::to_string(i), ypos, -1.);
        fast_output_options->get("zpos"+std::to_string(i), zpos, -1.);
        if (xpos<0. || ypos<0. || zpos<0.) {
          output.write("\tAdded %i fast_output points\n", i);
          break;
        }
        ix = int(xpos*mesh->GlobalNx);
        iy = int(ypos*mesh->GlobalNy);
        iz = int(zpos*mesh->GlobalNz);
        fast_output.add("n"+std::to_string(i), n, ix, iy, iz);
        fast_output.add("T"+std::to_string(i), T, ix, iy, iz);
        i++;
      }
    }

Output is written to one (or several, if necessary) separate netCDF files, named
``BOUT.fast.i.nc`` by default (where i is the index of the processor writing output).
