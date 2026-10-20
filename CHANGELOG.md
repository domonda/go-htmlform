# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

This package was the `htmlform` module of `github.com/domonda/go-function`
until its release `htmlform/v0.7.1`. Its earlier changes are documented in the
[go-function changelog](https://github.com/domonda/go-function/blob/main/CHANGELOG.md).

## [Unreleased]

### Added

- `htmlform.ArgValidator` validates a parsed form argument with a `func(ctx context.Context, value any) error`
  registered by `Handler.SetArgValidator`; it gets the request context, and its error text
  is shown to the client in the 400 Bad Request response.
- `htmlform.Handler.SetMaxRequestBodySize` limits form submissions including uploads per handler
  (default 100 MiB, a size <= 0 rejects every submission); larger requests are answered
  with 413 Request Entity Too Large, and uploads over `httpfun.MaxMultipartFormMemory`
  are buffered in temporary files instead of memory.
- `htmlform.Handler.SetCrossOriginProtection` configures, or with nil disables,
  the protection of form submissions against cross-site request forgery.
- `htmlform` takes pointers, slices and arrays of file types from uploaded files;
  a slice or array gets all files uploaded with the same field name,
  also files with the same name, which go-fs names like `scan (2).pdf`,
  and more files than an array holds are rejected with 400 Bad Request.

### Changed

- **Breaking:** the module path is `github.com/domonda/go-htmlform`
  instead of `github.com/domonda/go-function/htmlform`.
- **Breaking:** requires the v1.0.0 API of `github.com/domonda/go-function`, see its changelog.
- **Breaking:** requires `github.com/ungerik/go-fs` v1.0.0-beta.1 instead of a pseudo-version,
  which opens every upload by its own file and field name: names like `report%201.pdf`
  or with a `"` no longer answer 404, and a file named `..%2Fother%2Fx.txt` no longer reads
  the upload of the field `other`. `ReadAll` and `ReadAllString` of go-fs files take a context.
- **Breaking:** `htmlform.Handler.SetArgValidator` takes an `htmlform.ArgValidator`
  instead of `types.Validatable`, and htmlform no longer depends on `github.com/domonda/go-types`.
- **Breaking:** `htmlform.NewHandler` returns an error for arguments with a file type in a struct field
  or map, which uploads can't fill and a text value like `{"F":"/etc/hosts"}` set to a path on the server,
  and for file arguments that an uploaded `fs.File` can't be assigned to, like `fs.MemFile`
  or `*[]fs.File`, whose uploads failed with 500 Internal Server Error.
- **Breaking:** `htmlform.Handler.SetArgInputType` panics for `"file"` on an argument
  that doesn't take uploaded files, because the upload was ignored.
- **Breaking:** `htmlform.Handler` answers form submissions that browsers send from other origins
  with 403 Forbidden.
- **Breaking:** the `SetArg*` methods of `htmlform.Handler` panic for names
  that are not arguments of the wrapped function.
- **Breaking:** without an uploaded file, an `fs.FileReader` argument of `htmlform` is nil.
- `htmlform.Handler` parses every form value once and calls the function with these values,
  so validators check exactly the values the function gets.
- `Handler` answers an argument value the function can't parse with 400 Bad Request
  instead of 500 Internal Server Error, and a request body over the size limit
  with 413 Request Entity Too Large. The response names the argument
  but not its value or the parse error.

### Fixed

- `htmlform.Handler` fixes:
  - argument types that are recursive, like `type T []T`, no longer make every request loop forever;
  - `time.Duration` and other number types implementing `encoding.TextUnmarshaler`, like `slog.Level`,
    get a text field instead of a number field that can't submit values like `5s` or `INFO`;
  - default values are formatted so that a submitted form passes them again: pointers are dereferenced,
    nil is empty, `[]byte` is text, slices, maps and structs are JSON, types implementing
    `encoding.TextMarshaler` use it, and `time.Time` fits `date` and `datetime-local` inputs;
  - a nil `ResultsWriter` writes nothing like `httpfun.RespondNothing` instead of panicking after the call;
  - HEAD requests are answered like GET, and 405 responses have an `Allow` header;
  - the input type `"radio"` renders one radio button per option of `SetArgOptions`;
  - the form is buffered, so a template error is answered without a partly written form,
    and it has the content type `text/html; charset=utf-8` also for custom templates;
  - arguments named `_` get no form field and their zero value;
  - multiple values of a field are the elements of a slice or array argument;
  - a file array of length 1 doesn't allow selecting multiple files;
  - a variadic argument is not required;
  - `SetArgValidator` with a nil validator removes the validator instead of failing every submission.
- `htmlform` argument validators were never called; they now run on the parsed value
  before the function, and a failing validator answers 400 Bad Request naming the argument.
- Data race in `htmlform.Handler` where concurrent GET requests could render each other's form fields.
- `htmlform` no longer writes template or form parsing error messages into responses,
  and removes temporary files of uploads also when parsing the form failed.
- `htmlform` number fields of float arguments have `step="any"`, so browsers accept decimal values like `98.6`
  instead of only integers.
- `Handler` panics again with `http.ErrAbortHandler`, so that net/http aborts the response,
  instead of answering it with 500 Internal Server Error.
- The documentation states that required fields are only enforced by browsers.

### Security

- `htmlform` takes `fs.File` and `fs.FileReader` arguments only from uploaded files;
  a non-empty text value for such an argument is rejected with 400 Bad Request instead of reaching
  the function as a path of a server file. The documentation no longer recommends
  a global string to `fs.FileReader` scanner and warns against exposing file arguments
  through `httpfun` or `cli` string arguments.
- Slice and array literals with more than `function.MaxSliceLiteralElements` elements
  are answered with 400 Bad Request.
