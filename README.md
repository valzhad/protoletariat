# Protocol Buffers for the Rest of Us

[![CI](https://github.com/cpcloud/protoletariat/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/cpcloud/protoletariat/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/protoletariat)](https://pypi.org/project/protoletariat)

## Motivation

`protoletariat` has one goal: fixing the broken imports for the Python code
generated by `protoc`.

## Usage

Here's an example of how to use the tool, called `protol`:

1. Create a few protobuf files

```protobuf
// thing1.proto
syntax = "proto3";

import "thing2.proto";

package things;

message Thing1 {
  Thing2 thing2 = 1;
}
```

```protobuf
// thing2.proto
syntax = "proto3";

package things;

message Thing2 {
  string data = 1;
}
```

2. Run `protoc` on those files

```sh
$ mkdir out
$ protoc \
  --python_out=out \
  --proto_path=directory/containing/protos thing1.proto thing2.proto
```

3. Run `protol` on the generated code

```sh
$ protol \
  --create-package \
  --in-place \
  --python-out out \
  protoc --proto-path=directory/containing/protos thing1.proto thing2.proto
```

The `out/thing1_pb2.py` file should show a diff containing at least these lines:

```patch
-import thing2_pb2 as thing2__pb2
-
+from . import thing2_pb2 as thing2__pb2
```

## How it works

At a high level `protoletariat` converts absolute imports to relative imports.

However, it doesn't convert just any absolute import to a relative import.

The `protol` tool will only convert imports that were generated from `.proto` files. It
does this by inspecting `FileDescriptorProtos` from the protobuf files.

The core rewrite mechanism is implemented using a simplified form of pattern
matching, that looks at the Python AST, and if an import pattern is matched a
corresponding rewrite rule is invoked.

## Help

```
$ protol
Usage: protol [OPTIONS] COMMAND [ARGS]...

  Rewrite protoc or buf-generated imports for use by the protoletariat.

Options:
  -o, --python-out DIRECTORY      Directory containing protoc or buf-generated Python code  [required]
  --in-place / --not-in-place     Overwrite all relevant files under `--python-out` with adjusted imports  [default: not-in-place]
  --create-package / --dont-create-package
                                  Recursively create __init__.py files under `--python-out`  [default: dont-create-package]
  -s, --module-suffixes TEXT      Suffixes of Python/mypy modules to process  [default: _pb2.py, _pb2.pyi, _pb2_grpc.py, _pb2_grpc.pyi]
  --help                          Show this message and exit.

Commands:
  buf     Use buf to generate the FileDescriptorSet blob
  protoc  Use protoc to generate the FileDescriptorSet blob
```
