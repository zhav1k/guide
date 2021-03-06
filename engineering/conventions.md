# Development Conventions

* [Introduction](#introduction)
* [Operating System Support](#operating-system-support)
* [Versioning](#versioning)
* [Build System](#build-system)
* [Continuous Integration](#continuous-integration)
* [Publishing](#publishing)
* [Docker](#docker)
* [Error Handling](#error-handling)
* [TODO and FIXME comments](#todo-and-fixme-comments)
* [Others](#others)
* [Development Environments](#development-environments)
* [CLI](#cli)
* [Logging](#logging)
* [Configuration](#configuration)

## Introduction

This guide documents development conventions that are language-independent, so they are applicable to every source{d} project.

We have additional conventions for each language:

* [Go](conventions/go.md)
* [Python](conventions/python.md)
* [Scala](conventions/scala.md)

## Operating System Support

We support Linux, macOS and Windows on amd64. Support for other operating systems and architectures will require some help from the community. Expect support for, at least, two latest Ubuntu LTS releases, two latest macOS releases and latest Windows and Windows Server releases.

Some projects might define both build and runtime requirements that go beyond what is available by default on each operating system. Please, check the README of the project to know more about its requirements.

## Versioning

* Use [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html).
  * Note that backwards compatibility rules [do not apply to `0.y.z` versions](https://semver.org/#spec-item-4).
* When creating version tags in git, use the `v` prefix (e.g. `v0.5.0`). See our [git guide](git-flow.md) for more information.

## Build System

* All projects have a `Makefile` at the top-level directory.
* Makefile should be compatible with `[GNU make](https://www.gnu.org/software/make/manual/make.html)`.
* [src-d/ci](https://github.com/src-d/ci) should be used in these makefiles to provide integration with our continuous integration systems.

## Continuous Integration

* All open source projects should be integrated with [Travis CI](https://travis-ci.org/) for continuous integration. Check [src-d/ci examples](https://github.com/src-d/ci/tree/master/examples).
* Use [Appveyor](https://www.appveyor.com/) for CI on Windows. We support Windows unless it is not possible at all (e.g. [bblfshd](https://github.com/bblfsh/bblfshd)).
* Drone is used for continuous delivery. If you are working on a web application, read the [continuous delivery guide](continuous-delivery.md) too.

## Publishing

* Binaries should be attached to GitHub Releases. src-d/ci will do this automatically.

## Docker

* Applications should be packaged with Docker.
* Include `Dockerfile` in the top-level directory of the project.
* Use latest [alpine](https://hub.docker.com/_/alpine/) or [busybox](https://hub.docker.com/_/busybox/) whenever it is possible. Use latest [debian](https://hub.docker.com/_/debian/) stable-slim otherwise.
* If needed to reduce image size, use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/).

## Error Handling

* If it is idiomatic in your language, prefer early handling of errors and return early (more info: [Avoid Else, Return Early](http://blog.timoxley.com/post/47041269194/avoid-else-return-early), [GuardClause](http://wiki.c2.com/?GuardClause) and [HandleErrorsInContext](http://wiki.c2.com/?HandleErrorsInContext)).

## TODO and FIXME comments

TODO and FIXME comments should match the following format:

```
// KEYWORD(reference): comment text
```

A single line comment token must be used (`//` in case of C/C++, Go, etc). This is done to allow commenting large chunks of code with multi-line comment tokens (`/*`) during development and refactoring.

The `KEYWORD` must be one of the following:

- `TODO` - Highlights future work or unanswered questions that are outside the scope of the current PR, but that should be considered when modifying this code in the future. For example, a missing optimization, possible new features, something to refactor, or a deprecated usage to replace.
- `FIXME` - There is a known issue or bug which needs to be addressed. Those comments should only be committed if a new bug was discovered during working on other feature/bugfix, if the fix requires significant design changes or you don't have knowledge to address it. In other cases it's better to try fixing the bug, of course.

The `reference` must be either:

- A Github issue reference: `TODO(#123)`. This is the preferred reference format, especially for `FIXME` comments. An issue will help track the technical debt associated with `TODO/FIXME` comments and will start a discussions about new features or design considerations to address those comments.
- A Github username: `TODO(user)`. The author of the comment, or the person with the most context about the motivation for the comment (for example, a reviewer may request a TODO assigned to themselves). This reference type is most useful for `TODO` comments. Make sure to include a descriptive comment text, to help the reader understand the motivation without having to contact the comment author.

Multiline comments should be separated from regular comments either by a blank line:

```
// TODO(user): Special comment line 1.
// Special comment line 2.

// Regular comment.
```

or by a commented blank line:

```
// TODO(user): Special comment line 1.
// Special comment line 2.
//
// Regular comment.
```

If you choose to indent the comment body, do it consistently with spaces: 

```
// TODO(user): Special comment line 1.
//             Special comment line 2.
//
//             Special comment line 3.
//
// Regular comment.
```

## Others

Refer to the [licensing](licensing.md), [repositories](repositories.md) and [documentation](documentation.md) guides for other important information you should know when creating new projects.

## Development Environments

Each sourcerer has his own preferences on editors and IDEs. But if you do not know what to use, [Visual Studio Code](https://code.visualstudio.com/) is a popular choice for Go and Python, and [IntelliJ IDEA](https://www.jetbrains.com/idea/) is the most used for Java and Scala.

## CLI

Your application will expose one or more binaries with a CLI.

[Admin processes](https://12factor.net/admin-processes), such as initializing an environment, should use the same [configuration](#configuration). Usually they will be git-style subcommands of the main binary.

We prefer [GNU-style command line options](http://www.catb.org/esr/writings/taoup/html/ch10s05.html).
That is, double dash for long options and single dash for short options (single letter). Include `--help` and `--version`. As a general rule, define always long flags, and then optionally short flags for tools that are often used interactively. When defining flags it is worth to consider matching behavior of widely used flags. For example, if `--all` is present, `-a` should be its shorthand. `--quiet` as a default instead of `--silent`. `--file/-f` for a file argument, `--output/-o` for output path, etc.

## Logging

* Use structured logging.
* By default, output pretty logs when running in a terminal, output JSON when not.
* Allow changing logging level and format explicitly through environment variables (`LOG_LEVEL`, `LOG_FORMAT`).
* Use the following keys where they apply:
  * `time` for log timestamp in ISO 8601 format. Full timestamp with nanosecond resolution and offset is preferred, but lower resolutions are supported too.
  * `duration` for duration of an operation, when output is JSON, specify it as integer nanoseconds.
  * `level` for log level.
  * `error` for error message (optionally with stacktrace).
  * `msg` for main log message.
  * `source` for source file originating the log message (e.g. `foo.go:32`).

## Configuration

Your application should be [configured](https://12factor.net/config) using environment variables. External services (e.g. databases) URLs, storage paths, [ports to bind](https://12factor.net/port-binding) and logging format all belong to configuration.

Prefer the usage of URI to describe [backing services](https://12factor.net/backing-services). For example, prefer `DATABASE=mysql://host:port/db` or `BROKER=amqp://user:pass@broker:port` over three or more different settings.
