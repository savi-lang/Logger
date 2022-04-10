# Logger

An application-level logging library in the Savi standard library.

## Features

- High performance
  - Always-lazy log expression evaluation
    - Constant, near-zero overhead for non-emitted logs, even when the logged expression takes work to construct
  - No imposed pointer indirection or virtual calls
    - Minimal overhead in general, even with custom formatting
    - Easier for LLVM to optimize and inline all log filtering and formatting logic

- Shareable across actors
  - `Logger` is a `val`, so it can be shared across multiple actors.
  - No need for each actor to construct their own private `Logger`, though they can if needed.

- Capability-secure
  - A `Logger` cannot emit anything without being given a sink to emit into
    - This means nothing can log to stdout/stderr or a file without being explicitly granted the capability to do so

- Runtime log level filtering
  - Each `Logger` has its own filter level, assigned at runtime.

- Customizable types and formatting
  - a type parameter sets the log expression type
  - a type parameter sets the log sink type
  - a type parameter sets the log formatter module (which transforms the log expression type into the log sink type)

- Batteries-included
  - Useful log formatter modules are included out of the box.

## Best practices

### Log in the application, not in libraries

Prefer logging only in the main application codebase, or in purpose-written libraries maintained alongside the application.

Choices about logging (what to log, at what levels, and how to format it) should be made by the application team. Any choices that a library author makes may not match what the application team wants, or what other library authors chose, and the overall logging output may be a mess.

Instead, to be log-friendly, libraries should prefer writing their functions to be minimal and self-contained enough that placing logging outside of them is more appropriate than inside of them anyway.

An exception to this advice is made for libraries that act as opinionated *frameworks*, where the choices about logging are part of the opinions enforced by the framework. In such a case, it is quite expected that library choices will preempt application developer choices about application-level concerns, so logging in such a framework can be considered good practice.

### Use a type alias `Log` for your configured `Logger` type

A `Logger` is configured with type parameters, but you don't want to write those identical type parameters all across your application where the type is used.

Instead, you'll want to define it in one central place, with an alias like this:

```savi
:alias Log: Logger(MyCustomLogFormatterModule, MyCustomLogType)
```

In fact, this library is named `Logger` instead of `Log` primarily because we wanted to leave the name `Log` open to application use instead of taking it with this library. The above type alias pattern should be considered the standard way of using this library.

Thanks to this alias, everywhere in your application that needs to hold a logger, you can use the short type name `Log` instead of writing the verbose type name `Logger(MyCustomLogFormatterModule, MyCustomLogType)`. And if you ever need to change the configuration (like switching to a different log formatter module), then you can easily do so in one place.
