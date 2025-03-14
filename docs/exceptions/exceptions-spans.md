<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Spans
--->

# Semantic Conventions for Exceptions on Spans

**Status**: [Stable][DocumentStatus]

This document defines semantic conventions for recording application
exceptions associated with spans.

<!-- toc -->

- [Recording an Exception](#recording-an-exception)
- [Attributes](#attributes)
  - [Stacktrace Representation](#stacktrace-representation)

<!-- tocstop -->

## Recording an Exception

An exception SHOULD be recorded as an `Event` on the span during which it occurred.
The name of the event MUST be `"exception"`.

A typical template for an auto-instrumentation implementing this semantic convention
using an [API-provided `recordException` method](https://github.com/open-telemetry/opentelemetry-specification/tree/v1.31.0/specification/trace/api.md#record-exception)
could look like this (pseudo-Java):

```java
Span span = myTracer.startSpan(/*...*/);
try {
  // Code that does the actual work which the Span represents
} catch (Throwable e) {
  span.recordException(e, Attributes.of("exception.escaped", true));
  throw e;
} finally {
  span.end();
}
```

## Attributes

The table below indicates which attributes should be added to the `Event` and
their types.

<!-- semconv trace-exception -->
The event name MUST be `exception`.

| Attribute  | Type | Description  | Examples  | [Requirement Level](https://opentelemetry.io/docs/specs/semconv/general/attribute-requirement-level/) | Stability |
|---|---|---|---|---|---|
| [`exception.escaped`](../attributes-registry/exception.md) | boolean | SHOULD be set to true if the exception event is recorded at a point where it is known that the exception is escaping the scope of the span. [1] |  | `Recommended` | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`exception.message`](../attributes-registry/exception.md) | string | The exception message. | `Division by zero`; `Can't convert 'int' object to str implicitly` | See below | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`exception.stacktrace`](../attributes-registry/exception.md) | string | A stacktrace as a string in the natural representation for the language runtime. The representation is to be determined and documented by each language SIG. | `Exception in thread "main" java.lang.RuntimeException: Test exception\n at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)` | `Recommended` | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`exception.type`](../attributes-registry/exception.md) | string | The type of the exception (its fully-qualified class name, if applicable). The dynamic type of the exception should be preferred over the static type in languages that support it. | `java.net.ConnectException`; `OSError` | See below | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |

**[1]:** An exception is considered to have escaped (or left) the scope of a span,
if that span is ended while the exception is still logically "in flight".
This may be actually "in flight" in some languages (e.g. if the exception
is passed to a Context manager's `__exit__` method in Python) but will
usually be caught at the point of recording the exception in most languages.

It is usually not possible to determine at the point where an exception is thrown
whether it will escape the scope of a span.
However, it is trivial to know that an exception
will escape, if one checks for an active exception just before ending the span,
as done in the [example for recording span exceptions](#recording-an-exception).

It follows that an exception may still escape the scope of the span
even if the `exception.escaped` attribute was not set or set to false,
since the event might have been recorded at a time where it was not
clear whether the exception will escape.

**Additional attribute requirements:** At least one of the following sets of attributes is required:

* [`exception.type`](../attributes-registry/exception.md)
* [`exception.message`](../attributes-registry/exception.md)
<!-- endsemconv -->

### Stacktrace Representation

The table below, adapted from [Google Cloud][gcp-error-reporting], includes
possible representations of stacktraces in various languages. The table is not
meant to be a recommendation for any particular language, although SIGs are free
to adopt them if they see fit.

| Language   | Format                                                              |
| ---------- | ------------------------------------------------------------------- |
| C#         | the return value of [Exception.ToString()][csharp-stacktrace]       |
| Elixir     | the return value of [Exception.format/3][elixir-stacktrace]         |
| Erlang     | the return value of [`erl_error:format`][erlang-stacktrace]         |
| Go         | the return value of [runtime.Stack][go-stacktrace]                  |
| Java       | the contents of [Throwable.printStackTrace()][java-stacktrace]      |
| Javascript | the return value of [error.stack][js-stacktrace] as returned by V8  |
| Python     | the return value of [traceback.format_exc()][python-stacktrace]     |
| Ruby       | the return value of [Exception.full_message][ruby-full-message]     |

Backends can use the language specified methodology for generating a stacktrace
combined with platform information from the
[telemetry sdk resource][telemetry-sdk-resource] in order to extract more fine
grained information from a stacktrace, if necessary.

[gcp-error-reporting]: https://cloud.google.com/error-reporting/reference/rest/v1beta1/projects.events/report
[java-stacktrace]: https://docs.oracle.com/javase/7/docs/api/java/lang/Throwable.html#printStackTrace%28%29
[python-stacktrace]: https://docs.python.org/3/library/traceback.html#traceback.format_exc
[js-stacktrace]: https://v8.dev/docs/stack-trace-api
[ruby-full-message]: https://ruby-doc.org/core-2.7.1/Exception.html#method-i-full_message
[csharp-stacktrace]: https://docs.microsoft.com/dotnet/api/system.exception.tostring
[go-stacktrace]: https://pkg.go.dev/runtime/debug#Stack
[telemetry-sdk-resource]: ../resource/README.md#telemetry-sdk
[erlang-stacktrace]: https://www.erlang.org/doc/man/erl_error.html#format_exception-3
[elixir-stacktrace]: https://hexdocs.pm/elixir/1.14.3/Exception.html#format/3
[DocumentStatus]: https://github.com/open-telemetry/opentelemetry-specification/tree/v1.31.0/specification/document-status.md
