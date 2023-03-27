<!--
This README describes the package. If you publish this package to pub.dev,
this README's contents appear on the landing page for your package.

For information about how to write a good package README, see the guide for
[writing package pages](https://dart.dev/guides/libraries/writing-package-pages).

For general information about developing packages, see the Dart guide for
[creating packages](https://dart.dev/guides/libraries/create-library-packages)
and the Flutter guide for
[developing packages and plugins](https://flutter.dev/developing-packages).
-->
# transit-dart

Transit is a data format and set of libraries for conveying values between
applications written in different languages. This library provides support for
marshalling Transit data to/from Dart.

* [Transit Rationale](https://blog.cognitect.com/blog/2014/7/22/transit)
* [Transit Specification](https://github.com/cognitect/transit-format)
* [transit-dart API docs](https://pub.dev/documentation/transit_dart/latest/)

This implementation's major.minor version number corresponds to the version of
the Transit specification it supports.

_NOTE: Transit is intended primarily as a wire protocol for transferring data
between applications. If storing Transit data durably, readers and writers are
expected to use the same version of Transit and you are responsible for
migrating/transforming/re-storing that data when and if the transit format
changes._

## Releases and Dependency Information

* Latest release: 0.8.79 -- still in alpha, API is not fixed

## Getting started

```
dart pub add transit_dart
```

Then in your Dart code, you can use:

```dart
import 'package:transit_dart/transit_dart.dart';
```

See [Installing](https://pub.dev/packages/transit_dart/install).

## Usage

```dart
import 'package:collection/collection.dart';
import 'package:transit_dart/transit_dart.dart';

/// Encodes and decodes some objects. Note that although the objects are stored
/// in a list, they are encoded and decoded separately, each one treated as a
/// top-level object. This is expected behavior for transit and is an example of
/// how transit brokers in not just one, but _streams_ of JSON objects.
Future<void> main() async {
  // Some objects to work with.
  var objects = <dynamic>[
    "hello",
    ["A", "B", null, true, 3.4],
    {42: "the answer"}
  ];
  print('objects: $objects');

  // Encode the objects to a List<String>;
  var writer = TransitEncoder.json().fuse(JsonRepeatEncoder());
  var encoded = await Stream.fromIterable(objects).transform(writer).toList();
  print('encoded: ${encoded.join()}');

  // Decode the objects to a List<dynamic>
  var reader = JsonRepeatDecoder().fuse(TransitDecoder.json());
  var decoded = await Stream.fromIterable(encoded).transform(reader).toList();
  print('decoded: $decoded');

  // Did everything come back same as we sent it?
  var test = DeepCollectionEquality().equals(objects, decoded);
  print('Round trip success? ${test ? 'YES' : 'NO'}');
}
```

## Default Type Mapping

|Transit Type   |Write accepts           |Read produces           |
|------         |------                  |------                  |
|null           |null                    |null                    |
|string         |String                  |String                  |
|boolean        |bool                    |bool                    |
|integer        |int                     |int                     |
|decimal        |double                  |double                  |
|bytes          |Uint8List               |Uint8List               |
|keyword        |transit_dart.Keyword    |transit_dart.Keyword    |
|symbol         |transit_dart.Symbol     |transit_dart.Symbol     |
|big decimal    |transit_dart.BigDecimal (wraps big_decimal/BigDecimal)|transit_dart.BigDecimal (wraps big_decimal/BigDecimal)|
|big integer    |BigInt                  |BigInt                  |
|time           |DateTime                |DateTime                |
|uuid           |transit_dart.Uuid       |transit_dart.Uuid       |
|uri            |transit_dart.TransitUri |transit_dart.TransitUri |
|char           |String                  |String                  |
|special numbers|double.nan, double.infinity, double.negativeInfinity|double.nan, double.infinity, double.negativeInfinity|
|array          |List                    |List                    |
|map            |Map                     |Map                     |
|set            |Set                     |Set                     |
|list           |transit_dart.TransitList|transit_dart.TransitList|
|link           |transit_dart.Link       |transit_dart.Link       |

## Testing

To run the roundtrip verification tests in `transit-format`, first ensure
Dart>=2.19.1 and Java 8 are installed, then do the following:

1. Set up a testing directory where all this can take place. The
   `transit-format` library and `transit-dart` library need to be side-by-side
   under the same parent directory. For example, create `transit-test` and
   inside clone [transit-format](https://github.com/cognitect/transit-format).

```sh
mkdir transit-test
cd transit-test
git clone https://github.com/cognitect/transit-format.git
# Do something similar to the following to ensure Java 8.
jenv local 1.8
```

2. Tell `transit-format` that the dart version is supported. In file
   `src/transit/verify.clj`, near line 350, make this change:

```clj
;; in file `transit-format/src/transit/verify.clj`
(def supported-impls #{"transit-clj"
                       "transit-cljs"
                       "transit-dart"   ;<-- insert this line
                       "transit-java"
                       "transit-jruby"
                       "transit-js"
                       "transit-python"
                       "transit-ruby"})
```

3. Copy `get-transit-dart` from `transit-dart/bin` into `transit-format/bin`.

```sh
curl "https://raw.githubusercontent.com/wevre/transit-dart/master/bin/get-transit-dart" > transit-format/bin/get-transit-dart
chmod +x transit-format/bin/get-transit-dart
```

4. Execute the verify command. The `verify` command will check for and, as
   necessary, clone `transit-dart`, run `dart pub get`, and compile
   `roundtrip.dart`.

```
transit-format/bin/verify -impls dart
```

## Additional information

Coming soon, more info about the package: where to find more information, how to
contribute to the package, how to file issues, what response they can expect
from the package authors, and more.

## Copyright and License

Copyright © 2023 Michael J. Weaver

This library is a Dart port of the Java version created and maintained by
Cognitect.

Copyright © 2014 Cognitect

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at https://www.apache.org/licenses/LICENSE-2.0. Unless required by
applicable law or agreed to in writing, software distributed under the License
is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied. See the License for the specific language
governing permissions and limitations under the License.
