# LIBUSB\_ERROR\_BUSY

This error most often occurs when you try to connect to a ChipWhisperer
device that is already connected elsewhere (e.g. in another notebook).

You must disconnect from the first session, either with `scope.dis()` or
by restarting the notebook's kernel, before you can connect to ChipWhisperer
in another notebook.

It's also possible for this error to occur if Husky has gotten into an
unexpected state that it can't recover from (failed stream captures can
sometimes do this). In this case, a hard reset resolves the problem.

This applies to all ChipWhisperer hardware and software.

