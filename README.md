# hjson-go

[![Build Status](https://github.com/hjson/hjson-go/workflows/test/badge.svg)](https://github.com/hjson/hjson-go/actions)
[![Go Pkg](https://img.shields.io/github/release/hjson/hjson-go.svg?style=flat-square&label=go-pkg)](https://github.com/hjson/hjson-go/releases)
[![Go Report Card](https://goreportcard.com/badge/github.com/hjson/hjson-go?style=flat-square)](https://goreportcard.com/report/github.com/hjson/hjson-go)
[![coverage](https://img.shields.io/badge/coverage-ok-brightgreen.svg?style=flat-square)](https://gocover.io/github.com/hjson/hjson-go/)
[![godoc](https://img.shields.io/badge/godoc-reference-blue.svg?style=flat-square)](https://godoc.org/github.com/hjson/hjson-go/v4)

![Hjson Intro](https://hjson.github.io/hjson1.gif)

```
{
  # specify rate in requests/second (because comments are helpful!)
  rate: 1000

  // prefer c-style comments?
  /* feeling old fashioned? */

  # did you notice that rate doesn't need quotes?
  hey: look ma, no quotes for strings either!

  # best of all
  notice: []
  anything: ?

  # yes, commas are optional!
}
```

The Go implementation of Hjson is based on [hjson-js](https://github.com/hjson/hjson-js). For other platforms see [hjson.github.io](https://hjson.github.io).

More documentation can be found at https://pkg.go.dev/github.com/hjson/hjson-go/v4

# Install

Make sure you have a working Go environment. See the [install instructions](https://golang.org/doc/install.html).

- In order to use Hjson from your own Go source code, just add an import line like the one here below. Before building your project, run `go mod tidy` in order to download the Hjson source files. The suffix `/v4` is required in the import path, unless you specifically want to use an older major version.
```go
import "github.com/hjson/hjson-go/v4"
```
- If you instead want to use the **hjson-cli** command line tool, run the command here below in your terminal. The executable will be installed into your `go/bin` folder, make sure that folder is included in your `PATH` environment variable.
```bash
$ go install github.com/hjson/hjson-go/hjson-cli@latest
```
# Usage as command line tool
```
usage: hjson-cli [OPTIONS] [INPUT]
hjson can be used to convert JSON from/to Hjson.

hjson will read the given JSON/Hjson input file or read from stdin.

Options:
  -bracesSameLine
      Print braces on the same line.
  -c  Output as JSON.
  -h  Show this screen.
  -indentBy string
      The indent string. (default "  ")
  -j  Output as formatted JSON.
  -omitRootBraces
      Omit braces at the root.
  -quoteAlways
      Always quote string values.
  -v
      Show version.
```

Sample:
- run `hjson-cli test.json > test.hjson` to convert to Hjson
- run `hjson-cli -j test.hjson > test.json` to convert to JSON

# Usage as a GO library

```go

package main

import (
    "github.com/hjson/hjson-go/v4"
    "fmt"
)

func main() {
    // Now let's look at decoding Hjson data into Go
    // values.
    sampleText := []byte(`
    {
        # specify rate in requests/second
        rate: 1000
        array:
        [
            foo
            bar
        ]
    }`)

    // We need to provide a variable where Hjson
    // can put the decoded data.
    var dat map[string]interface{}

    // Decode with default options and check for errors.
    if err := hjson.Unmarshal(sampleText, &dat); err != nil {
        panic(err)
    }
    // short for:
    // options := hjson.DefaultDecoderOptions()
    // err := hjson.UnmarshalWithOptions(sampleText, &dat, options)
    fmt.Println(dat)

    // In order to use the values in the decoded map,
    // we'll need to cast them to their appropriate type.

    rate := dat["rate"].(float64)
    fmt.Println(rate)

    array := dat["array"].([]interface{})
    str1 := array[0].(string)
    fmt.Println(str1)


    // To encode to Hjson with default options:
    sampleMap := map[string]int{"apple": 5, "lettuce": 7}
    hjson, _ := hjson.Marshal(sampleMap)
    // short for:
    // options := hjson.DefaultOptions()
    // hjson, _ := hjson.MarshalWithOptions(sampleMap, options)
    fmt.Println(string(hjson))
}
```

## Unmarshal to Go structs

If you prefer, you can also unmarshal to Go structs (including structs implementing the json.Unmarshaler interface or the encoding.TextUnmarshaler interface). The Go JSON package is used for this, so the same rules apply. Specifically for the "json" key in struct field tags. For more details about this type of unmarshalling, see the [documentation for json.Unmarshal()](https://pkg.go.dev/encoding/json#Unmarshal).

```go

package main

import (
    "github.com/hjson/hjson-go/v4"
    "fmt"
)

type Sample struct {
    Rate  int
    Array []string
}

type SampleAlias struct {
    Rett    int      `json:"rate"`
    Ashtray []string `json:"array"`
}

func main() {
    sampleText := []byte(`
    {
        # specify rate in requests/second
        rate: 1000
        array:
        [
            foo
            bar
        ]
    }`)

    // unmarshal
    var sample Sample
    hjson.Unmarshal(sampleText, &sample)

    fmt.Println(sample.Rate)
    fmt.Println(sample.Array)

    // unmarshal using json tags on struct fields
    var sampleAlias SampleAlias
    hjson.Unmarshal(sampleText, &sampleAlias)

    fmt.Println(sampleAlias.Rett)
    fmt.Println(sampleAlias.Ashtray)
}
```

## Comments on struct fields

By using key `comment` in struct field tags you can specify comments to be written on one or more lines preceding the struct field in the Hjson output.

```go

package main

import (
    "github.com/hjson/hjson-go/v4"
    "fmt"
)

type foo struct {
    A string `json:"x" comment:"First comment"`
    B int32  `comment:"Second comment\nLook ma, new lines"`
    C string
    D int32
}

func main() {
    a := foo{A: "hi!", B: 3, C: "some text", D: 5}
    buf, err := hjson.Marshal(a)
    if err != nil {
        fmt.Error(err)
    }

    fmt.Println(string(buf))
}
```

Output:

```
{
  # First comment
  x: hi!

  # Second comment
  # Look ma, new lines
  B: 3

  C: some text
  D: 5
}
```

# API

[![godoc](https://godoc.org/github.com/hjson/hjson-go/v4?status.svg)](https://godoc.org/github.com/hjson/hjson-go/v4)

# History

[see releases](https://github.com/hjson/hjson-go/releases)
