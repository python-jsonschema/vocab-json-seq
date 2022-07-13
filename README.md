# A JSON Schema Vocabulary for JSON Text Sequences (RFC 7464)

## 1. Purpose

This document describes a simple [JSON Schema vocabulary](https://json-schema.org/draft/2020-12/json-schema-core.html#name-schema-vocabularies) that can be used to validate JSON text sequences as specified by [RFC 7464](https://datatracker.ietf.org/doc/html/rfc7464).

It defines two keywords which allow applying a JSON schema to individual elements in a JSON text sequence (hereafter "sequence") and producing an annotation of the element-wise result of application of the schema.

Though indeed the entirety of a sequence is not itself a standard JSON type (nor valid JSON), proscribed below is a loose [`stream` type](#streams) whose implementation is left mostly to the specific language or implementation to further define.

## 2. Declarations

The ID for this vocabulary is `https://python-jsonschema.github.io/vocab-json-seq/` (the URI to this document).

A [draft 2020-12](https://json-schema.org/specification-links.html#2020-12) meta-schema which includes this vocabulary has been defined for convenience.
The `$id` for the meta-schema is `https://python-jsonschema.github.io/vocab-json-seq/meta.json`, and it can also be found at this address.

## 3. The `jsonseq` Keyword

### 3.1 Syntax and Semantics

The `jsonseq` keyword is a JSON Schema [annotation](https://json-schema.org/draft/2020-12/json-schema-core.html#name-annotations) whose value MUST be a valid JSON Schema.

Applying the keyword to a stream instance MUST produce a single annotation result, itself a new stream.
The contents of the annotation stream MUST be the corresponding result of applying the schema element-wise to each element of the sequence.

Validating an empty stream against this keyword (one which contains no elements) produces an empty stream annotation, as does validating a non-stream instance.
(As discussed [below](#

### 3.2 Contextual Behavior

The `jsonseq` keyword MUST be processed contextually in accordance with the draft of the schema in which it is used. For example, if `jsonseq` is used in a schema that declares draft 2019-09, then its schema value must be processed using the rules specified by the 2019-09 specification.

### 3.3 Beyond JSON Text Sequences

Implementations MAY choose to offer support for notionally similar formats to JSON Text Sequence, such as [`jsonl`](https://jsonlines.org/), which uses newlines instead of record separators but is otherwise quite similar.

If such support is present, implementations SHOULD use the same `jsonseq` keyword to apply validation to streams containing `jsonl` data.

## 4. Streams

### 4.1 Overview & Representation

The processing of the `jsonseq` keyword, or truthfully of JSON Text Sequences themselves, depend on the abstract notion of a "stream".

This document does not define the specific implementation of streams. A programming language or implementation with lazy iterable support SHOULD represent streams using this language feature.

Implementations MUST also consider JSON `array` values to be streams for the purpose of the keywords defined in this vocabulary.

Schema authors who do not wish to allow `array` valued instances are RECOMMENDED to use existing JSON Schema mechanisms to exclude them (e.g. `{"not": {"type": "array"}}`).

The core JSON vocabulary [does not allow](https://json-schema.org/draft/2020-12/json-schema-core.html#name-instance-data-model) external vocabularies to define additional data types via the `type` keyword.
It does however [allow](https://json-schema.org/draft/2020-12/json-schema-core.html#name-non-json-instances) for the application of JSON Schema to types beyond those provided by JSON.
A `streamType` keyword is therefore introduced below, which can be used to assert a value is a stream in the sense defined here.

(Editor's note: the definition of `streamType` may be moved to a separate vocabulary in the future).

### 4.2 The `streamType` Keyword

The value of the `streamType` keyword MUST be a boolean, or the value `null`.

When `true`, validation MUST succeed if the instance is a stream, and fail otherwise.
When `false`, validation MUST fail if the instance is a stream, and succeed otherwise.
When `null`, validation always succeeds.

## 5. A Short Example

Consider the following schema, utilizing the two keywords above, which asserts that elements of a sequence are objects whose `foo` property is an integer at most 10:

```json
{
  "$schema": "https://python-jsonschema.github.io/vocab-json-seq/meta.json",
  "streamType": true,
  "jsonseq": {
    "type": "object",
    "properties": {
      "foo": {
        "type": "integer",
        "maximum": 10,
      }
    }
  }
}
```

For simplicity, we delimit elements of a JSON Text Sequence below using newlines.
Consider the sequence:

```
{}
{}
{"foo": 12}
{"foo": 8}
{"foo": {}}
{"foo": 1}
{}
```

The aforementioned schema, when applied to this sequence, should produce a corresponding sequence of validation results:

```
true
true
false
true
false
true
true
```

## 6. Limitations & Open Questions

Implementations of JSON Schema which do not support types beyond those present in JSON will undoubtedly not be able to implement this vocabulary easily.
In particular, implementation of this vocabulary requires a statically-typed implementation's `validate` entry point to operate on a union type (of JSON or stream), or requires equivalent language functionality.
An interesting note is that such possibilities may not be unique to this vocabulary, as any vocabulary introducing a new non-JSON-native type may change the signature of their validation methods.
