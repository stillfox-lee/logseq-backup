title:: Go 1.20 Is Released!
author:: [[The Go Blog]]
url:: https://go.dev/blog/go1.20

- > Go’s type conversion rules have been extended to permit direct conversion [from a slice to an array](https://go.dev/ref/spec#Conversions_from_slice_to_array_or_array_pointer). ([View Highlight](https://read.readwise.io/read/01gr8ceyyrppcrpre9me9zaxj8))
- > The new function [`errors.Join`](https://go.dev/pkg/errors#Join) returns an error wrapping a list of errors which may be obtained again if the error type implements the `Unwrap() []error` method. ([View Highlight](https://read.readwise.io/read/01gr8cfn560mz3yd4ns5ephbsg))
- > The new [`context.WithCancelCause`](https://go.dev/pkg/context#WithCancelCause) function provides a way to cancel a context with a given error. That error can be retrieved by calling the new [`context.Cause`](https://go.dev/pkg/context#Cause) function. ([View Highlight](https://read.readwise.io/read/01gr8cganrnk8zdcmpf8qqsw45))
- > We’re particularly excited to launch a preview of [profile-guided optimization](https://go.dev/doc/pgo) (PGO), which enables the compiler to perform application- and workload-specific optimizations based on run-time profile information. Providing a profile to `go build` enables the compiler to speed up typical applications by around 3–4%, and we expect future releases to benefit even more from PGO. Since this is a preview release of PGO support, we encourage folks to try it out, but there are still rough edges which may preclude production use. ([View Highlight](https://read.readwise.io/read/01gr8c2waa6vxt9r6qwq1k6f3r))
	- **Tags**: #[[golang]]
	- **Date**: 2023-02-02