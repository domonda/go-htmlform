# HTTP status codes reference

This page lists every response status that [`htmlform.Handler`](https://pkg.go.dev/github.com/domonda/go-htmlform#Handler) produces, the body it writes, and the cause. The errors of the function are mapped like for [`httpfun.Handler`](https://pkg.go.dev/github.com/domonda/go-function/httpfun#Handler), see the [HTTP status codes reference of go-function](https://github.com/domonda/go-function/blob/main/docs/reference-http-status-codes.md).

## htmlform.Handler

| Status | Body | Cause |
|---|---|---|
| 200 OK | the HTML form | GET or HEAD |
| 200 OK | written by the `ResultsWriter` | POST, the function returned without error |
| 400 Bad Request | `invalid multipart form` | the POST body is not a valid `multipart/form-data` form, including URL-encoded forms |
| 400 Bad Request | `invalid value for argument "a"` | the value of field `a` couldn't be parsed |
| 400 Bad Request | `invalid argument "a": <validator error>` | the validator set with `SetArgValidator("a", ...)` returned an error |
| 400 Bad Request | `argument "a" must be an uploaded file` | a non-empty text value was sent for a file argument |
| 400 Bad Request | `argument "a" takes at most N files` | more files than an array argument `[N]fs.File` holds |
| 403 Forbidden | `Forbidden` | a browser sent the POST from another origin, see `SetCrossOriginProtection` |
| 200 OK | empty | POST, the function returned without error and the `ResultsWriter` is nil |
| 405 Method Not Allowed | `Method Not Allowed` | any method other than GET, HEAD, and POST, with the header `Allow: GET, HEAD, POST` |
| 413 Request Entity Too Large | `Request Entity Too Large` | the body exceeded the limit set by `SetMaxRequestBodySize`, default 100 MiB |
| any, 404, 500 | as for `httpfun.Handler` | the function returned an error, mapped like the `httpfun.Handler` rows of the go-function reference |
| 500 Internal Server Error | `Internal Server Error` | the function panicked, always recovered |
| 500 Internal Server Error | `Internal Server Error` | a temporary file for an upload couldn't be written, or the form template failed |

Notes:

- The 400 errors for parsing, validation, and files, and the errors of the function, go to the `ResultsWriter` first, as its `resultErr` argument. A custom writer can render them, for example by showing the form again with the message. The built-in writers return them, and `httpfun.HandleError` writes the response.
- The 400, 403, 405, and 413 responses for the form body itself (`invalid multipart form`, cross-origin, method, size) are written directly and skip the `ResultsWriter` and `HandleError`.
- The function is not called for the 400, 403, 405, and 413 responses in the table. It can still cause a 4xx response itself by returning an error like `httperr.NotFound` or one wrapping `os.ErrNotExist`.
- File arguments that uploads can't fill, like `fs.MemFile` or a file type in a struct field, are rejected by `NewHandler` before any request.

## Related

- [HTTP status codes reference of go-function](https://github.com/domonda/go-function/blob/main/docs/reference-http-status-codes.md): `httpfun.Handler` and how an error becomes a response
- [Security model of go-function](https://github.com/domonda/go-function/blob/main/docs/explanation-security-model.md): why responses hide argument values and internal errors
