# jsoncons: a C++ library for json construction

jsoncons is a C++ library for the construction of [JavaScript Object Notation (JSON)](http://www.json.org). It supports parsing a JSON file or string into a `json` value, building a `json` value in C++ code, and serializing a `json` value to a file or string. It also provides an API for generating json read and write events in code, somewhat analogously to SAX processing in the XML world. 

jsoncons uses some features that are new to C++ 11, including [move semantics](http://thbecker.net/articles/rvalue_references/section_02.html) and the [AllocatorAwareContainer](http://en.cppreference.com/w/cpp/concept/AllocatorAwareContainer) concept. It has been tested with MS VC++ 2013, MS VC++ 2015, GCC 4.8, GCC 4.9, GCC 6.2.0 and recent versions of clang. Note that `std::regex` isn't fully implemented in GCC 4.8., so `jsoncons_ext/jsonpath` regular expression filters aren't supported for that compiler.

The [code repository](https://github.com/danielaparker/jsoncons) and [releases](https://github.com/danielaparker/jsoncons/releases) are on github. It is distributed under the [Boost Software License](http://www.boost.org/users/license.html)

Features:

- Uses the standard C++ input/output streams library
- Supports conversion from and to the standard library sequence and associative containers
- Supports conversion from and to user defined types
- Passes all tests from [JSON_checker](http://www.json.org/JSON_checker/) except `fail1.json`, which is allowed in [RFC7159](http://www.ietf.org/rfc/rfc7159.txt)
- Returns the expected results for all tests from [JSONTestSuite](https://github.com/nst/JSONTestSuite)
- Supports object members sorted alphabetically by name or in original order
- Implements parsing and serializing JSON text in UTF-8 for narrow character strings and streams
- Supports UTF16 (UTF32) encodings with size 2 (size 4) wide characters
- Correctly handles surrogate pairs in \uXXXX escape sequences
- Supports event based JSON parsing and serializing with user defined input and output handlers
- Accepts and ignores single line comments that start with //, and multi line comments that start with /* and end with */
- Parses files with duplicate names but uses only the last entry
- Supports optional escaping of the solidus (/) character
- Supports Nan, Inf and -Inf replacement
- Supports reading a sequence of JSON texts from a stream
- Supports optional escaping of non-ascii UTF-8 octets
- Supports conversion from and to user defined types
- Handles JSON texts of arbitrarily large depth of nesting, a limit can be set if desired

Extensions:

- The [jsonpath](#user-content-ext_jsonpath) extension supports search using [Stefan Goessner's JsonPath](http://goessner.net/articles/JsonPath/).  It also supports search and replace using JsonPath expressions.
- The [csv](#user-content-ext_csv) extension supports reading (writing) JSON values from (to) CSV files
- The [msgpack](#user-content-ext_msgpack) extension supports encoding to and decoding from the [MessagePack](http://msgpack.org/index.html) binary serialization format.

## What's new on master

A bug was introduced in 0.99.7 causing the values of existing object members to not be changed with set or assignment operations. This has been fixed on master and in Release 0.99.7.2.

## Get jsoncons

The jsoncons library is header-only: it consists solely of header files containing templates and inline functions, and requires no separately-compiled library binaries when linking. It has no dependence on other libraries. The accompanying test suite uses boost, but not the library itself.

To install the jsoncons library, download the zip file, unpack the release, under `src` find the directory `jsoncons`, and copy it to your `include` directory. If you wish to use extensions, copy the `jsoncons_ext` directory as well. 

## How to use it

- For a quick guide, see [jsoncons: a C++ library for json construction](http://danielaparker.github.io/jsoncons). 
- Consult the [tutorials and reference](https://github.com/danielaparker/jsoncons/wiki) for the details. 

As the `jsoncons` library has evolved, names have sometimes changed. To ease transition, jsoncons deprecates the old names but continues to support many of them. See the [deprecated list](https://github.com/danielaparker/jsoncons/wiki/deprecated) for the status of old names. The deprecated names can be suppressed by defining macro `JSONCONS_NO_DEPRECATED`, which is recommended for new code.

## Benchmarks

[json_benchmarks](https://github.com/danielaparker/json_benchmarks) provides some measurements about how `jsoncons` compares to other `json` libraries.
Results for [JSONTestSuite](https://github.com/nst/JSONTestSuite) and [JSON_checker](http://www.json.org/JSON_checker/) tests may be found [here](https://danielaparker.github.io/json_benchmarks/).

## A simple program using jsoncons

```c++
#include <iostream>
#include <jsoncons/json.hpp>

// For convenience
using jsoncons::json;

int main()
{
    json color_spaces = json::array();
    color_spaces.add("sRGB");
    color_spaces.add("AdobeRGB");
    color_spaces.add("ProPhoto RGB");

    json image_sizing; // empty object
    image_sizing["Resize To Fit"] = true; // a boolean 
    image_sizing["Resize Unit"] = "pixels"; // a string
    image_sizing["Resize What"] = "long_edge"; // a string
    image_sizing["Dimension 1"] = 9.84; // a double
    
    json file_export;

    // create "File Format Options" as an object and put "Color Spaces" in it
    file_export["File Format Options"]["Color Spaces"] = std::move(color_spaces); 

    file_export["Image Sizing"] = std::move(image_sizing);

    std::cout << "(1)\n" << pretty_print(file_export) << "\n\n";

    const json& val = file_export["Image Sizing"];

    std::cout << "(2) " << "Dimension 1 = " << val["Dimension 1"].as<double>() << "\n\n";

    std::cout << "(3) " << "Dimension 2 = " << val.get_with_default("Dimension 2","n/a") << "\n";
}
```
Output:
```json
(1)
{
    "File Format Options": {
        "Color Spaces": ["sRGB","AdobeRGB","ProPhoto RGB"]
    },
    "Image Sizing": {
        "Dimension 1": 9.84,
        "Resize To Fit": true,
        "Resize Unit": "pixels",
        "Resize What": "long_edge"
    }
}

(2) Dimension 1 = 9.84

(3) Dimension 2 = n/a
```

## Building the test suite and examples with CMake

[CMake](https://cmake.org/) is a cross-platform build tool that generates makefiles and solutions for the compiler environment of your choice. On Windows you can download a [Windows Installer package](https://cmake.org/download/). On Linux it is usually available as a package, e.g., on Ubuntu,
```
sudo apt-get install cmake
```

Instructions for building the test suite with CMake may be found in

    jsoncons/test_suite/build/cmake/README.txt

Instructions for building the examples with CMake may be found in

    jsoncons/examples/build/cmake/README.txt

## About jsoncons::basic_json

The jsoncons library provides a `basic_json` class template, which is the generalization of a `json` value for different character types, different policies for ordering name-value pairs, etc.
```c++
typedef basic_json<char,
                   JsonTraits = json_traits<char>,
                   Allocator = std::allocator<char>> json;
```
The library includes four instantiations of `basic_json`:

- [json](https://github.com/danielaparker/jsoncons/wiki/json) constructs a narrow character json value that sorts name-value members alphabetically

- [ojson](https://github.com/danielaparker/jsoncons/wiki/ojson) constructs a narrow character json value that preserves the original name-value insertion order

- [wjson](https://github.com/danielaparker/jsoncons/wiki/wjson) constructs a wide character json value that sorts name-value members alphabetically

- [wojson](https://github.com/danielaparker/jsoncons/wiki/wojson) constructs a wide character json value that preserves the original name-value insertion order

## Features

### Meaningful error messages

```c++
try 
{
    json val = json::parse("[1,2,3,4,]");
} 
catch(const std::exception& e) 
{
    std::cout << e.what() << std::endl;
}
```
Output:
```
Extra comma at line 1 and column 10
```
### Range-based for loops with arrays

```c++
json j = json::array{1,2,3,4};

for (auto val : book.array_range())
{
    std::cout << val << std::endl;
}
```

### Range-based for loops with objects

```c++
json book = json::object{
    {"author", "Haruki Murakami"},
    {"title", "Kafka on the Shore"},
    {"price", 25.17}
};

for (const auto& kvp : book.object_range())
{
    std::cout << kvp.key() << "=" 
              << kvp.value() << std::endl;
}
```
### _json and _ojson literal operators

```c++
json j1 = R"(
{
    "StartDate" : "2017-03-01",
    "MaturityDate" : "2020-12-30"          
}
)"_json;

ojson j2 = R"(
{
    "StartDate" : "2017-03-01",
    "MaturityDate" : "2020-12-30"          
}
)"_ojson;
```

### Multi-dimensional json arrays
```c++
json a = json::make_array<3>(4, 3, 2, 0.0);
double val = 1.0;
for (size_t i = 0; i < a.size(); ++i)
{
    for (size_t j = 0; j < a[i].size(); ++j)
    {
        for (size_t k = 0; k < a[i][j].size(); ++k)
        {
            a[i][j][k] = val;
            val += 1.0;
        }
    }
}
std::cout << pretty_print(a) << std::endl;
```
Output:
```json
[
    [
        [1.0,2.0],
        [3.0,4.0],
        [5.0,6.0]
    ],
    [
        [7.0,8.0],
        [9.0,10.0],
        [11.0,12.0]
    ],
    [
        [13.0,14.0],
        [15.0,16.0],
        [17.0,18.0]
    ],
    [
        [19.0,20.0],
        [21.0,22.0],
        [23.0,24.0]
    ]
]
```
See [Arrays](https://github.com/danielaparker/jsoncons/wiki/Arrays) for details

### Convert from and to standard library sequence containers

```c++
std::vector<int> v{1, 2, 3, 4};
json j(v);
std::cout << "(1) "<< j << std::endl;
std::deque<int> d = j.as<std::deque<int>>();
```
Output:
```
(1) [1,2,3,4]
```

### Convert from and to standard library associative containers

```c++
std::map<std::string,int> m{{"one",1},{"two",2},{"three",3}};
json j(m);
std::cout << "(1) " << j << std::endl;
std::unordered_map<std::string,int> um = j.as<std::unordered_map<std::string,int>>();
```
Output:
```
(1) {"one":1,"three":3,"two":2}
```

### Convert from and to user defined types (and standard library containers of user defined types)

```c++
struct book
{
    std::string author;
    std::string title;
    double price;
};

namespace jsoncons
{
    template<class Json>
    struct json_type_traits<Json, book>
    {
        // Implement static functions is, as and to_json 
    };
}        

book book1{"Haruki Murakami", "Kafka on the Shore", 25.17};
book book2{"Charles Bukowski", "Women: A Novel", 12.0};

std::vector<book> v{book1, book2};

json j = v;

std::list<book> l = j.as<std::list<book>>();
```

See [Type Extensibility](https://github.com/danielaparker/jsoncons/wiki/Type%20Extensibility) for details.

### Filter json names and values

You can rename object member names with the built in filter [rename_name_filter](https://github.com/danielaparker/jsoncons/wiki/rename_name_filter)

```c++
#include <sstream>
#include <jsoncons/json.hpp>
#include <jsoncons/json_filter.hpp>

using namespace jsoncons;

int main()
{
    std::string s = R"({"first":1,"second":2,"fourth":3,"fifth":4})";    

    json_serializer serializer(std::cout);

    // Filters can be chained
    rename_name_filter filter2("fifth", "fourth", serializer);
    rename_name_filter filter1("fourth", "third", filter2);

    // A filter can be passed to any function that takes
    // a json_input_handler ...
    std::cout << "(1) ";
    std::istringstream is(s);
    json_reader reader(is, filter1);
    reader.read();
    std::cout << std::endl;

    // or a json_output_handler    
    std::cout << "(2) ";
    ojson j = ojson::parse(s);
    j.dump(filter1);
    std::cout << std::endl;
}
```
Output:
```json
(1) {"first":1,"second":2,"third":3,"fourth":4}
(2) {"first":1,"second":2,"third":3,"fourth":4}
```
Or define and use your own filters. See [json_filter](https://github.com/danielaparker/jsoncons/wiki/json_filter) for details.

## Extensions

<div id="ext_jsonpath"/>

### jsonpath

Example file (store.json):
```json
{ "store": {
    "book": [ 
      { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      },
      { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      },
      { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      }
    ]
  }
}
```
```c++
#include <jsoncons/json.hpp>
#include <jsoncons_ext/jsonpath/json_query.hpp>

using namespace jsoncons;
using namespace jsoncons::jsonpath;

int main()
{
    std::ifstream is("input/booklist.json");
    json booklist;
    is >> booklist;

    // All books whose author's name starts with Evelyn
    json result1 = json_query(booklist, "$.store.book[?(@.author =~ /Evelyn.*?/)]");
    std::cout << "(1)\n" << pretty_print(result1) << std::endl;

    // Change the price of "Moby Dick"
    json_replace(booklist,"$.store.book[?(@.isbn == '0-553-21311-3')].price",10.0);
    std::cout << "(2)\n" << pretty_print(booklist) << std::endl;

}
```
Output:
```json
(1)
[
    {
        "author": "Evelyn Waugh",
        "category": "fiction",
        "price": 12.99,
        "title": "Sword of Honour"
    }
]
(2)
{
    "store": {
        "book": [
            {
                "author": "Nigel Rees",
                "category": "reference",
                "price": 8.95,
                "title": "Sayings of the Century"
            },
            {
                "author": "Evelyn Waugh",
                "category": "fiction",
                "price": 12.99,
                "title": "Sword of Honour"
            },
            {
                "author": "Herman Melville",
                "category": "fiction",
                "isbn": "0-553-21311-3",
                "price": 10.0,
                "title": "Moby Dick"
            }
        ]
    }
}
```

See [json_query](https://github.com/danielaparker/jsoncons/wiki/json_query), [json_replace](https://github.com/danielaparker/jsoncons/wiki/json_replace), and [Basics](https://github.com/danielaparker/jsoncons/wiki/Basics) for details.

<div id="ext_csv"/>

### csv
Example file (tasks.csv)
```csv
project_id, task_name, task_start, task_finish
4001,task1,01/01/2003,01/31/2003
4001,task2,02/01/2003,02/28/2003
4001,task3,03/01/2003,03/31/2003
4002,task1,04/01/2003,04/30/2003
4002,task2,05/01/2003,
```

```c++
#include <fstream>
#include <jsoncons/json.hpp>
#include <jsoncons_ext/csv/csv_reader.hpp>
#include <jsoncons_ext/csv/csv_serializer.hpp>

using namespace jsoncons;
using namespace jsoncons::csv;

int main()
{
    std::ifstream is("input/tasks.csv");

    json_decoder<json> decoder;
    csv_parameters params;
    params.assume_header(true)
          .trim(true)
          .ignore_empty_values(true) 
          .column_types({"integer","string","string","string"});
    csv_reader reader(is,decoder,params);
    reader.read();
    ojson tasks = decoder.get_result();

    std::cout << "(1)\n" << pretty_print(tasks) << "\n\n";

    std::cout << "(2)\n";
    csv_serializer serializer(std::cout);
    tasks.dump(serializer);
}
```
Output:
```json
(1)
[
    {
        "project_id": 4001,
        "task_name": "task1",
        "task_start": "01/01/2003",
        "task_finish": "01/31/2003"
    },
    {
        "project_id": 4001,
        "task_name": "task2",
        "task_start": "02/01/2003",
        "task_finish": "02/28/2003"
    },
    {
        "project_id": 4001,
        "task_name": "task3",
        "task_start": "03/01/2003",
        "task_finish": "03/31/2003"
    },
    {
        "project_id": 4002,
        "task_name": "task1",
        "task_start": "04/01/2003",
        "task_finish": "04/30/2003"
    },
    {
        "project_id": 4002,
        "task_name": "task2",
        "task_start": "05/01/2003"
    }
]
```
```csv
(2)
project_id,task_name,task_start,task_finish
4001,task2,02/01/2003,02/28/2003
4001,task3,03/01/2003,03/31/2003
4002,task1,04/01/2003,04/30/2003
4002,task2,05/01/2003,
```

See [csv_reader](https://github.com/danielaparker/jsoncons/wiki/csv_reader) and [csv_serializer](https://github.com/danielaparker/jsoncons/wiki/csv_serializer) for details.

<div id="ext_msgpack"/>

### msgpack

The `msgpack` extension supports encoding to and decoding from the [MessagePack](http://msgpack.org/index.html) binary serialization format.

#### MessagePack example

Example file (book.json):
```json
[
    {
        "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
    },
    {
        "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
    }
]
```
```c++
#include <jsoncons/json.hpp>
#include <jsoncons_ext/msgpack/message_pack.hpp>

using namespace jsoncons;
using namespace jsoncons::msgpack;

int main()
{
    std::ifstream is("input/book.json");
    ojson j1;
    is >> j1;

    // Encode ojson to MessagePack
    std::vector<uint8_t> v = encode_message_pack(j1);

    // Decode MessagePack to ojson 
    ojson j2 = decode_message_pack<ojson>(v);

    std::cout << pretty_print(j2) << std::endl;

    // or to json (now alphabetically sorted)
    json j3 = decode_message_pack<json>(v);

    // or to wjson (converts from utf8 to wide characters)
    wjson j4 = decode_message_pack<wjson>(v);
}
```
Output:
```json
[
    {
        "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
    },
    {
        "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
    }
]
```

See [encode_message_pack](https://github.com/danielaparker/jsoncons/wiki/encode_message_pack) and [decode_message_pack](https://github.com/danielaparker/jsoncons/wiki/decode_message_pack) for details.

## Acknowledgements

Special thanks to our [contributors](https://github.com/danielaparker/jsoncons/blob/master/acknowledgements.txt)

