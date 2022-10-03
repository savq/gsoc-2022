# Final Report on An Alternative to Distributed for Pluto.jl

Sergio A. Vargas \
email: <savargasqu+git@unal.edu.co> \
GitHub: [@savq](https://github.com/savq)


### Introduction

During the 2022 Google Summer of Code I worked on developing an alternative package to Distributed.jl
for the Pluto.jl notebook environment. The motivation for the work can be found in more detail in the
[proposal for the project.](https://github.com/savq/gsoc-2022/blob/master/proposal.md)


### Malt.jl

Malt.jl —the main product of this project— is a multiprocessing package for Julia.
You can use Malt to create Julia processes, and to perform computations in those processes.
Unlike the standard library package Distributed.jl, Malt is focused on process sandboxing, not distributed computing.
The [source code for Malt is available on GitHub](https://github.com/JuliaPluto/Malt.jl),
and the [documentation is hosted on GitHub pages.](https://juliapluto.github.io/Malt.jl/stable/)

Malt currently supports almost all the features that Pluto requires to work properly.
However, An issue with Windows support for Julia (or rather, libuv a dependency of Julia)
means that Malt still doesn't fully support Windows.
The issue in question is the same as [JuliaLang/julia#44253](https://github.com/JuliaLang/julia/issues/44253).

I made multiple attempts to circumvent this issue. However, none proved to be sufficiently reliable or caused issues with other components of Malt.

It's unlikely that this issue can be fully solved _in_ Malt,
so the remaining work to do would be to fully fix the issue in libuv or Julia Base,
or to allow Pluto to keep using Distributed on Windows.


### Pluto on Malt

The second part of the project was integrating Malt into Pluto.
This task proved to be much easier than expected.
The Pluto codebase is well modularized.
In particular, most functions that actually call the external processes are in the Workspace Manager module,
so integrating Malt was simply replacing the calls to Distributed functions with calls to Malt,
and updating the tests accordingly.

The work on integrating Malt in Pluto is on [fonsp/Pluto.jl#2240](https://github.com/fonsp/Pluto.jl/pull/2240).

The only feature that Malt currently doesn't provide is to evaluate functions calls in the same process making the call.
In this case, instead of sending a request to another process, the call would be evaluated in the same process.
While this isn't very useful in general, removing the overhead of the remote calls speeds up testing quite a bit.

The issue of Malt on Windows and the issue of local evaluation can both be solved by defining an "evaluator" interface,
and writing three implementations:
1. A Malt implementation would be the default on Linux and Mac.
2. A Distributed implementation would be the default on Windows.
3. A "local evaluator" would be optional and would be used mainly for testing.

This interface can be very small, with only about 6 functions being necessary.
Since the API for Malt is modeled after the API for Distributed the implementations would also be very similar.
Defining the interface is effectively making explicit the work that was already done to swap Distributed for Malt.


### Malt beyond Pluto

Since the first proposal of the project,
I emphasized the need for the Distributed alternative to be its own stand-alone package.
The main reason is that being able to execute code in different processes is useful
in many contexts outside cluster computing (the main use case of Distributed).

Malt could be extended to allow non-Julia processes to create Malt workers, something that's out of scope for Distributed.
For example, This could allow editors and IDEs to create a Malt worker for code evaluation
(similar to Clojure's [nREPL](https://nrepl.org/nrepl/index.html)).
It would also allow Malt workers to be called from the shell
(similar to [DaemonMode.jl](https://github.com/dmolina/DaemonMode.jl)).

