<p align="center">
    <img src="Assets/CodableCSV.svg" alt="Codable CSV"/>
</p>

[CodableCSV](https://github.com/dehesa/CodableCSV) allows you to read and write CSV files row-by-row or through Swift's Codable interface.

![Swift 5.1](https://img.shields.io/badge/Swift-5.1-orange.svg) ![platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20macOS%20%7C%20tvOS%20%7C%20watchOS%20%7C%20Linux-lightgrey.svg) [![License](http://img.shields.io/:license-mit-blue.svg)](http://doge.mit-license.org) [![Build Status](https://travis-ci.com/dehesa/CodableCSV.svg?branch=master)](https://travis-ci.com/dehesa/CodableCSV)

This framework provides:

-   Row-by-row CSV reader & writer.
-   Codable interface.
-   Support for multiple inputs/outputs (in-memory, file system, binary socket, etc.).
-   CSV encoding & configuration inference (e.g. what field/row delimiters are being used).
-   Multiplatform support with no dependencies.

# Usage

To use this library, you need to add it to you project (through SPM or Cocoapods) and import it.

```swift
import CodableCSV
```

There are two ways to use [CodableCSV](https://github.com/dehesa/CodableCSV):

1. as an active row-by-row and field-by-field reader or writer.
2. through Swift's `Codable` interface.

> `CodableCSV` can _encode to_ or _decode from_ `String`s, `Data` blobs, or CSV files (represented by `URL` addresses).

## Active Encoding/Decoding

The _active entities_ provide _imperative_ control on how to read or write CSV data.

<ul>
<details><summary><code>CSVReader</code>.</summary><p>

A `CSVReadder` parses CSV data from an input and returns you each CSV row as an array of strings.

-   Row-by-row parsing.

    ```swift
    let reader = try CSVReader(data: ...)
    while let row = try reader.parseRow() {
        // Do something with the row: [String]
    }
    ```

-   `Sequence` syntax parsing.

    ```swift
    let reader = try CSVReader(fileURL: ...)
    for row in reader {
        // Do something with the row: [String]
    }
    ```

    Please note the `Sequence` syntax (i.e. `IteratorProtocol`) doesn't throw errors; therefore if the CSV data is invalid, the previous code will crash your program. If you don't control the origin of the CSV data, use the `parseRow()` function instead.

-   Whole input parsing.

    ```swift
    let file = try CSVReader.parse(string: ..., configuration: ...)
    // file is of type: (headers: [String], rows: [[String]])
    ```

### Reader Configuration

`CSVReader` accepts the following configuration properties:

-   `encoding` (default: `nil`) specify the CSV file encoding.

    This `String.Encoding` value specify how each underlying byte is represented (e.g. `.utf8`, `.utf32littleEndian`, etc.). If it is `nil`, the library will try to figure out the file encoding through the file's [Byte Order Marker](https://en.wikipedia.org/wiki/Byte_order_mark). If the file doesn't contain a BOM, `.utf8` is presumed.

-   `delimiters` (default: `(field: ",", row: "\n")`) specify the field and row delimiters.

    CSV fields are separated within a row with _field delimiters_ (commonly a "comma"). CSV rows are separated through _row delimiters_ (commonly a "line feed"). You can specify any unicode scalar, `String` value, or `nil` for unknown delimiters.

-   `headerStrategy` (default: `.none`) indicates whether the CSV data has a header row or not.

    CSV files may contain an optional header row at the very beginning. This configuration value lets you specify whether the file has a header row or not, or whether you want the library to figure it out.

-   `trimStrategy` (default: empty set) trims the given characters at the beginning and end of each parsed field.

    The trim characters are applied for the escaped and unescaped fields.

-   `presample` (default: `false`) indicates whether the CSV data should be completely loaded into memory before parsing begins.

    Loading all data into memory may provide faster iteration for small to medium size files, since you get rid of the overhead of managing an `InputStream`.

The configuration values are only set during initialization and can be passed to the `CSVReader` instance through a structure or with a convenience closure syntax:

```swift
let reader = CSVReader(data: ...) {
    $0.encoding = .utf8
    $0.delimiters.row = "\r\n"
    $0.headerStrategy = .firstLine
    $0.trimStrategy = .whitespaces
}
```

</p></details>

<details><summary><code>CSVWriter</code>.</summary><p>

#warning("Complete me")

</p></details>
</ul>

## Swift's `Codable`

The encoders/decoders provided by this library let you use Swift's `Codable` declarative approach to encode/decode CSV data.

<ul>
<details><summary><code>CSVDecoder</code>.</summary><p>

`CSVDecoder` transforms CSV data into a Swift type conforming to `Decodable`. The decoding process is very simple and it only requires creating a decoding instance and call its `decode` function passing the `Decodable` type and the input data.

```swift
let decoder = CSVDecoder()
let result = try decoder.decode(CustomType.self, from: data)
```

### Decoder Configuration

The decoding process can be tweaked by specifying configuration values at initialization time. `CSVDecoder` accepts the [same configuration values as `CSVReader`](#Reader-Configuration) plus the following ones:

-   `floatStrategy` (default: `.throw`) defines how to deal with non-conforming floating-point numbers (such as `NaN`, or `+Infinity`).

-   `decimalStrategy` (default: `.locale(nil)`) indicates how decimal numbers are decoded (from `String` to `Decimal` value).

-   `dataStrategy` (default: `.deferredToDate`) specify the strategy to use when decoding dates.

-   `dataStrategy` (default: `.base64`) specify the strategy to use when decoding data blobs.

-   `bufferingStrategy` (default: `.keepAll`) tells the decoder how to cache previously decoded CSV rows.

    Caching rows allow random access through `KeyedDecodingContainer`s.

The configuration values can be set during `CSVDecoder` initialization or at any point before the `decode` function is called.

```swift
let decoder = CSVDecoder {
    $0.encoding = .utf8
    $0.delimiters.field = "\t"
    $0.headerStrategy = .firstLine
    $0.bufferingStrategy = .ordered
}

decoder.decimalStratey = .custom {
    let value = try Float(from: $0)
    return Decimal(value)
}
```

</p></details>

<details><summary><code>CSVEncoder</code>.</summary><p>

#warning("Complete me")

</p></details>
</ul>

## Tips Using `Codable`

`Codable` is fairly easy to use and most Swift standard library types already conform to it. However, sometimes it is tricky to get custom types to comply to `Codable` for very specific functionality. That is why I am leaving here some tips and advices concerning its usage:

<ul>
<details><summary>Basic adoption.</summary><p>

`Codable` is just a type alias for `Decodable` and `Encodable`. When a custom type conforms to `Codable`, the type is stating that it has the ability to decode itself from and encode itself to a external representation. Which representation depends on the decoder or encoder chosen. Foundation provides support for [JSON and Property Lists](https://developer.apple.com/documentation/foundation/archives_and_serialization), but the community provide many other formats, such as: [YAML](https://github.com/jpsim/Yams), [XML](https://github.com/MaxDesiatov/XMLCoder), [BSON](https://github.com/OpenKitten/BSON), and CSV (through this library).

Lets see a regular CSV encoding/decoding usage through `Codable`'s interface. Let's suppose we have a list of students formatted in a CSV file:

```swift
let data = """
name,age,hasPet
John,22,true
Marine,23,false
Alta,24,true
"""
```

In Swift, a _student_ has the following structure:

```swift
struct Student: Codable {
    var name: String
    var age: Int
    var hasPet: Bool
}
```

To decode the CSV data, we just need to create a decoder and call `decode` on it passing the given data.

```swift
let decoder = CSVDecoder { $0.headerStrategy = .firstLine }
let students = try decoder.decode([Student], from: data)
```

The inverse process (from Swift to CSV) is very similar (and simple).

```swift
let encoder = CSVEncoder { $0.headerStraty = .firstLine }
let newData = try encoder.encode(students)
```

</p></details>

<details><summary>Specific behavior for CSV data.</summary><p>

When encoding/decoding CSV data, it is important to keep several points in mind:

</p>
<ul>
<details><summary>Default behavior requires a CSV with a headers row.</summary><p>

The default behavior (i.e. not including `init(from:)` and `encode(to:)`) rely on the existance of the synthesized `CodingKey`s whose `stringValue`s are the property names. For these properties to match any CSV field, the CSV data must contain a _headers row_ at the very beginning. If your CSV doesn't contain a _headers row_, you can specify coding keys with integer values representing the field index.

```swift
struct Student: Codable {
    var name: String
    var age: Int
    var hasPet: Bool

    private CodingKeys: Int, CodingKey {
        case name = 0
        case age = 1
        case hasPet = 2
    }
}
```

</p></details>
<details><summary>A CSV is a long list of records/rows.</summary><p>

CSV formatted data is commonly used with flat hierarchies (e.g. a list of students, a list of car models, etc.). Nested structures, such as the ones found in JSON files, are not supported by default in CSV implementations (e.g. a list of users, where each user has a list of services she uses, and each service has a list of the user's configuration values).

You can definitely support complex structures in CSV, but you would have to flatten the hierarchy in a single model or build a custom encoding/decoding process. This process would make sure there is always a maximum of two keyed/unkeyed containers.

As an example, we can create a nested structure for a school with students who own pets.

```swift
struct School: Codable {
    let students: [Student]
}

struct Student: Codable {
    var name: String
    var age: Int
    var pet: Pet
}

struct Pet: Codable {
    var nickname: String
    var gender: Gender

    enum Gender: Codable {
        case male, female
    }
}
```

By default the previous example wouldn't work. If you want to keep the nested structure, you need to overwrite the custom `init(from:)` implementation (to support `Decodable`).

```swift
extension School {
    init(from decoder: Decoder) throws {
        var container = try decoder.unkeyedContainer()
        while !container.isAtEnd {
            self.student.append(try container.decode(Student.self))
        }
    }
}

extension Student {
    init(from decoder: Decoder) throws {
        var container = try decoder.container(keyedBy: CustomKeys.self)
        self.name = try container.decode(String.self, forKey: .name)
        self.age = try container.decode(Int.self, forKey: .age)
        self.pet = try decoder.singleValueContainer.decode(Pet.self)
    }
}

extension Pet {
    init(from decoder: Decoder) throws {
        var container = try decoder.container(keyedBy: CustomKeys.self)
        self.nickname = try container.decode(String.self, forKey: .nickname)
        self.gender = try container.decode(Gender.self, forKey: .gender)
    }
}

extension Pet.Gender {
    init(from decoder: Decoder) throws {
        var container = try decoder.singleValueContainer()
        self = try container.decode(Int.self) == 1 ? .male : .female
    }
}

private CustomKeys: Int, CodingKey {
    case name = 0
    case age = 1
    case nickname = 2
    case gender = 3
}
```

You could have avoided building the initializers overhead by defining a flat structure such as:

```swift
struct Student: Codable {
    var name: String
    var age: Int
    var nickname: String
    var gender: Gender

    enum Gender: Int, Codable {
        case male = 1
        case female = 2
    }
}
```

</p></details>
</ul>

</details>

<details><summary>Configuration values and encoding/decoding strategies.</summary><p>

#warning("Complete me")

</p></details>

<details><summary>Performance advices.</summary><p>

#warning("Complete me")

</p></details>
</ul>

# Roadmap

<p align="center">
<img src="Assets/Roadmap.svg" alt="Roadmap"/>
</p>
