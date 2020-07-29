# Wrapcheck

A simple Go linter to check that errors from external packages are wrapped
during return to help identify the error source during debugging.

## Install

```bash
$ go get github.com/tomarrell/wrapcheck/cmd/wrapcheck
```

## Usage

To lint all the packages in a program:

```bash
$ wrapcheck ./...
```

## Why?

Errors in Go are simple values. They contain no more information about than the
minimum to satisfy the interface:

```go
type Error interface {
  Error() string
}
```

This is a fantastic feature, but can also be a limitation. Specifically when you
are attempting to identify the source of an error in your program.

As of Go 1.13, error wrapping using `fmt.Errorf(...)` is the recommend way to
compose errors in Go in order to add additional information.

Errors generated by your own code are usually predictable. However, when you
have a few frequently used libraries (think `sqlx` for example), you may run
into the dilemma of identifying exactly where in your program these errors are
caused.

In other words, you want a call stack.

This is especially apparent if you are a diligent Gopher and always hand your
errors back up the call stack, logging at the top level.

So how can we solve this?

## Solution

Wrapping errors at the call site.

When we call into external libraries which may return an error, we can wrap the
error to add additional information about the call site.

e.g.

```go
...

func (db *DB) createUser(name, email, city string) error {
  sql := `INSERT INTO customer (name, email, city) VALUES ($1, $2, $3);`

  if _, err := tx.Exec(sql, name, email, city); err != nil {
    // %v verb preferred to prevent error becoming part of external API
    return fmt.Errorf("failed to insert user: %v", err)
  }

  return nil
}

...
```

This solution allows you to add context which will be handed to the caller,
making identifying the source easier during debugging.

