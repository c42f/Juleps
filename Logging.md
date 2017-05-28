# JULEP Logging

- **Title:** A unified logging interface
- **Author:** Chris Foster <chris42f@gmail.com>
- **Created:** February 2017
- **Status:** work in progress

## Abstract

*Logging* is a tool for understanding program execution by recording the order and
timing of a sequence of events.  A *logging library* provides tools to define
these events in the source code and capture the event stream when the program runs.
The information captured from each event makes its way through the system as a
*log record*. The ideal logging library should give developers and users insight
into the running of their software by provide tools to filter, save and
visualize these records.

Julia has included simple logging in `Base` since version 0.1, but the tools to
generate and capture events are still immature as of version 0.6.  Because of
this, Julia 0.6 packages use any of several incompatible logging libraries, and
there's no systematic way to generate and capture log messages.

This proposal aims to fix this by defining the tools necessary to generate log
events

* Encourage package authors to include logging by keeping use simple but making
  it efficient and convenient.
* Add the minimum number of features and concepts required to support filtering
  and dispatch of log events in complex production environments.
* Unify logging under a common interface to avoid inconsistency of logging
  configuration and handling between packages.

## Desirable features

The desirable features fit into three rough categories - simplicity, flexibility
and efficiency.

Logging should be a *tool for package authors*.  It should be **simple to use** 

an average package author If log records are very easy to
generate, and should provide the average package author with as much insight
into t
and logged information should be as readable as possible by default.

and so that package authors can reach for
`info` rather than `println()`.  Logged information should be as readable as
possible presented in a way
which is highly readable .  We should have:

* A minimum of syntax - ideally just a logger verb and the message in most
  cases.  Context information for log messages (file name, line number, module,
  stack trace, etc.) should be automatically gathered without a syntax burden.
* Freedom in formatting the log message - simple string interpolation,
  `@sprintf` and `fmt()`, etc should all be fine.
* No mention of log dispatch should be necessary at the message creation site.
* Easy filtering of log messages.
* Clear guidelines about the meaning and appropriate use of standard log levels
  for consistency between packages, along with guiding the appropriate use of
  logging vs stdout.
* The default console log handler should integrate somehow with the display
  system to show log records in a way which is highly readable.  The idea is
  to make logging useful to package authors more encourage the use of logging over ad hoc console output.
  emphasize the *log message*, and
  metadata should be communicated in a non-intrusive way.


Logging should be a *tool for production *.

The API should be **flexible enough** for advanced users.


* **Log records** are more than a string: loggers typically gather context
  information both lexically (eg, module, file name, line number) and
  dynamically (eg, time, stack trace, thread id).  The API should preserve this
  structured information.
* Users should be able to add structured information to log records, to be
  preserved along with data extracted from the logging context. For example, a
  list of `key=value` pairs offers a decent combination of simplicity and power.
* Formatting and dispatch of log records should be in the hands of the user if
  they need it. For example, a user may want to write json records across the
  network to a log server.

* **Log dispatch** 

* For all packages using the standard logging API, it should be simple to
  intercept, filter and redirect logs in a unified and centrally controlled way.
* When unit testing a function, it should be possible to conveniently harvest
  any log records which are generated.
* Exceptions generated during log record creation shouldn't be able to crash the
  application.  This makes it safer to enable little-used logging code in a
  production system.
* 
* It should be possible to log to a user defined log context; automatically
  choosing a context for zero setup logging may not suit all cases.  For
  example, in some cases we may want to use a log context explicitly attached to
  a user-defined data structure.

* It should be possible to control log filtering per thread or task.
* 
    * Unique message IDs based on code location for finer grained message
      filtering?

The design should allow for an **efficient implementation**, to encourage
the availability of logging in production systems; logs you don't see should be
almost free, and logs you do see should be cheap to produce. The runtime cost
comes in three flavours:

* Cost in the logging library, to determine whether to filter a message.
* Cost in user code, to construct quantities which will only be used in the
  log message.
* Cost in the logging library in collecting context information and
  to dispatch and format log records.


## Proposed design

A prototype implementation is available at https://github.com/c42f/MicroLogging.jl

Key design issues -

FIXME: What+why -> decision / action for each issue

### Quickstart Example

#### Frontend
```julia
using Base.Log

# Logging macros
@debug "A message for debugging (filtered out by default)"
@info "Information about normal program operation"
@warn "A potentially problem was detected"
@error "Something definitely went wrong, but we recovered enough to continue"
@logmsg Log.Info "Explicitly defined info log level"

# Free form message formatting
x = 10.50
@info "$x"
@info @sprintf("%.3f", x)
@info begin
    A = ones(4,4)
    "sum(A) = $(sum(A))"
end

# Progress reporting
for i=1:10
    @info "Some algorithm" progress=i/10
end

# User defined key value pairs
foo_val = 10.0
@info "test" foo=foo_val bar=42
```


### What is a log record?

Logging statements are used to understand algorithm flow - the order and timing
in which logging events happen - and the program state at each event.  Each
logging event is preserved in a **log record**.  The information in a record
needs to be gathered efficiently, but should be rich enough to give insight into
program execution.

A log record includes information explicitly given at the call site, and any
relevant metadata which can be harvested from the lexical and dynamic
environment.  Most logging libraries allow for two key pieces of information
to be supplied explicitly:

* The **log message** - a user-defined string containing key pieces of program
  state, chosen by the developer.
* The **log level** - a category for the message, usually ordered from verbose
  to severe.  The log level is generally used as an initial filter to remove
  verbose messages.

Some logging libraries (for example
[glib](https://developer.gnome.org/glib/stable/glib-Message-Logging.html)
structured logging) allow users to supply extra log record information in the
form of key value pairs.  Others like
[log4j2](https://logging.apache.org/log4j/2.x/manual/messages.html) require extra information to be
explicitly wrapped in a log record type.  In julia, supporting key value pairs
in logging statements gives a good mixture of usability and flexibility:
Information can be communicated to the logging backend as simple keyword
function arguments, and the keywords provide syntactic hints for early filtering
in the logging macro frontend.

In addition to the explicitly provided information, some useful metadata can be
automatically extracted and stored with each log record.  Some of this is
extracted from the lexical environment or generated by the logging frontend
macro, including code location (module, file, line number) and a unique message
identifier.  The rest is dynamic state which can generally on demand by the
backend, including system time, stack trace, current task id.

### The logging frontend

The logging frontend is the interface used for generating log messages.  As
shown in the quickstart section, we use macros for this


### Dispatching records

What and why?

Records must be passed to some piece of code for dispatch.  Examples of dispatch
include formatting and printing to a terminal, saving it to a text file, sending
to a log server.

How?
* Global + context information
* 

### Early filtering


### User-defined log record fields

Examples of metadata

* Log levels
* 




## Concrete use cases

### Base

In Base, there are three somewhat disparate mechanisms for controlling logging.
An improved logging interface should unify these in a way which is convenient
both in the code and for user control.

* The 0.6 logging system's `logging()` function with redirection based on module
  and function.
* The `DEBUG_LOADING` mechanism in loading.jl and `JULIA_DEBUG_LOADING`
  environment variable.
* The depwarn system, and `--depwarn` command line flag


## Survey of other logging systems

We'll need a summary of how this all fits with the current logging needs of the
Julia ecosystem, as represented in Base, Logging.jl, MiniLogging.jl, Memento.jl,
LumberJack.jl, and other Julia libraries which do incidental logging, such as
Http.jl.

* [log4j2](https://logging.apache.org/log4j/2.x/) seems to be the current
  standard in Java logging. The design contains the lessons of twenty years of
  large systems. This means there's a fair amount of complexity and a lot of
  moving parts. However, the pieces are well documented and well designed.
* The python [logging library](https://docs.python.org/3/library/logging.html),
  originally described in [PEP-282](https://www.python.org/dev/peps/pep-0282/)
* glib (C) - https://developer.gnome.org/glib/stable/glib-Message-Logging.html
* a-cl-logger (Common lisp) - https://github.com/AccelerationNet/a-cl-logger
* Lager (Erlang) - https://github.com/erlang-lager/lager
* [RFC5424](https://datatracker.ietf.org/doc/rfc5424/?include_text=1) (The Syslog protocol)

TODO - much more here.

