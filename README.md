# A JSON Schema Vocabulary for JSON Text Sequences (RFC 7464)

## 1. Purpose

This document describes a simple [JSON Schema vocabulary](https://json-schema.org/draft/2020-12/json-schema-core.html#name-schema-vocabularies) defining two keywords that can be used to validate in the JSON Text Sequence format as specified by [RFC 7464](https://datatracker.ietf.org/doc/html/rfc7464).

In order to do so, it also loosely defines a `stream` type, representing elements of the sequence.

The purpose of keywords introduced are to allow the application of JSON Schema keywords to individual elements in a JSON text sequence (hereafter "sequence"), though indeed the entirety of a sequence is not itself JSON, nor does JSON Schema even have a "stream" type.

## 2. Declarations

The ID for this vocabulary is `https://python-jsonschema.github.io/vocab-json-seq/` (the URI to this document).

A [draft 2020-12](https://json-schema.org/specification-links.html#2020-12) meta-schema which includes this vocabulary has been defined for convenience.
The `$id` for the meta-schema is `https://python-jsonschema.github.io/vocab-json-seq/meta.json`, and it can also be found at this address.

## 3. The `jsonseq` Keyword

### 3.1 Syntax and Semantics

The value of the `jsonseq` keyword MUST be a valid JSON Schema.
This schema, when applied to a sequence, MUST be evaluated against each sequence element, thereby producing itself a stream of validation results corresponding to each element.

### 3.2 Contextual Behavior

The `jsonseq` keyword MUST be processed contextually in accordance with the draft of the schema in which it is used. For example, if `jsonseq` is used in a schema that declares draft 2019-09, then its schema value must be processed using the rules specified by the 2019-09 specification.

## 4. Streams

### 4.1 Overview & Representation

The processing of the `jsonseq` keyword, or truthfully of JSON Text Sequences themselves, depend on the abstract notion of a "stream".

This document does not define the specific implementation of streams. A programming language or implementation with lazy iterable support SHOULD represent streams using this language feature.

The core JSON vocabulary does not allow external vocabularies to define additional types via the `type` keyword.
Instead, introduced here below is a `streamType` keyword, which can be used to assert a value is a stream.

### 4.2 The `streamType` Keyword

The value of the `streamType` keyword MUST be a boolean, or the value `null`.

When `true`, validation MUST succeed if the instance is a stream, and fail otherwise.
When `false`, validation MUST fail if the instance is a stream, and succeed otherwise.
When `null`, validation always succeeds.

Implementations MUST also allow JSON `array` values to be considered streams for the purpose of the keywords defined in this vocabulary.

Schema authors who do not wish to allow `array` inputs are RECOMMENDED to use existing JSON Schema mechanisms to exclude them (e.g. `{"not": {"type": "array"}}`).

(Editor's note: the definition of `streamType` may be moved to a separate vocabulary in the future).

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
