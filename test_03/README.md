# Testing Paths

See `src/Makefile` for testing with some paths. If you want local libraries in
a `lib` folder, you need to specify the path in the Makefile where I define the
`LIBS` macro.

Otherwise, it assumes some "global" library. But, we might have to adjust paths
somehow.

Edit: if you include the extra argument to `LIBS`, it will compile correctly.
Note that this requires installing `libboost-all-dev`, and not `libboost-dev`
as I had earlier. Oops ...
