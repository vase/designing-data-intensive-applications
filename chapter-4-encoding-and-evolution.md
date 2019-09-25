# Chapter 4 - Encoding and Evolution
Link to someone's upload of the original chapter: https://notes.shichao.io/dda/ch4/
On how computers and programs write and send each other and themselves messages.
<details>
<summary>Why can't computers just write letters to each other?</summary>

- information representation on disk is not easily mirrored by plain text (pointers, references etc)
</details>

Other words for encoding:
- Serialization
- Marshalling

Other words for decoding:
- Parsing
- Deserialization
- Unmarshalling

(The book uses the term encoding, many resources on the internet use the term serialization. This chapter is about data serialization, not character encoding.)

## Why do we need encoding?
We want to move information from one place to another. So to and from:

- storage
- one application domain to another

This includes to and from processes on another computer, whether through removable physical storage, or networks. 

Often at the cost of human readability, different encodings allow us to use representations that preserve data accurately (including large floating numbers, datatypes), allow for manipulations of data that plain text would not allow ([streaming bytes of a huge file](https://stackoverflow.com/questions/6066774/how-does-file-streaming-actually-work)), or access/manipulate data quickly. 

### Some concrete examples
- Saving games
- XMLHttpRequests (or other requests)
- package.json

## How do we do encoding?

### What are the most common encoding methods, and how do they do their thing?
Non-language-specific:
  - Human readable(non-binary):
    <details>
    <summary>Good things</summary>
    
    - Popular ones can be consumed and produced by many programming languages
    - Human readable is good for debugging and for working with non-programmers
    - Flexible to add on to (no need for schema, does not require same structure to be parsed)
    </details>
    <details>
    <summary>Bad things</summary>
    
    - May not preserve information as well as other (e.g. numbers > 2^53 in JSON)
    - Limited to certain datatypes
    - Sometimes fragile (CSV and commas)
    - Slower to parse and bulkier
    - While schemas are available, tools may not bother using them; hardcoding interpretation based on schema may be required.
    </details>
    
    - JSON
    - XML
    - CSV

  - Binary:
    <details>
    <summary>Good things</summary>
    
    - Faster to parse, less bulky. Benefits scale with size of data storage/transmission.
    - Allows manipulations at the byte level
    - More accurate preservation of data
    </details>
    <details>
    <summary>Bad things</summary>
    
    - Humans can't read encoded data
    - Require schemas (and updates to schemas when data is changed)
    - May require [code generation](https://en.wikipedia.org/wiki/Code_generation_(compiler)) for statically typed languages (Conversely, allows for type checking at compile time while human-readable encoding doesn't)
    - May require special drivers to encode/decode (especially if data is obfuscated to protect proprietary secrets - conversely this may mean implementations are kept secure)
    </details>
    
    - Apache Thrift
    - Apache Avro
    - Protobuf
    - MessagePack
    
Many of the above binary encoding methods use type annotations, length indicators, and/or a field tag, plus the data from the entry itself. The field tags are tied to the keys of the original data, either manually (Thrift or Protocol Buffers) or programmatically. Fields may be specified to be required.
  ```
  {
      "userName": "Martin",
      "favoriteNumber": 1337,
      "interests": ["daydreaming", "hacking"]
  }
  ```

  converted to MessagePack, which is using type annotations and length indicators but including the original keys in binary form.

  ![MessagePack encoding](https://notes.shichao.io/dda/figure_4-1_600.png)
  
  converted to Protobuf, which uses both type annotations and field tags:
  
  ![Protobuf encoding](https://notes.shichao.io/dda/figure_4-4_600.png)
  
  and the accompanying schema:
  ```
  message Person {
    required string user_name = 1;
    optional int64 favorite_number = 2;
    repeated string interests = 3;
  }
  ```

  converted to Avro, which uses none of the above, instead relying on specifying the length of bytes to tell when an entry begins and ends:

  ![Avro encoding](https://notes.shichao.io/dda/figure_4-5_600.png)

  and the accompanying schema:
  ```
  record Person {
    string               userName;
    union { null, long } favoriteNumber = null;
    array<string>        interests;
  }
  ```

Language-specific
<details>
<summary>Issues (p113-114 of book)</summary>

- Reliance on programming language
- Needs to be able to instantiate arbitrary classes (vulnerability)
- Efficiency and backwards and forwards compatibility may be an issue
</details>

### Considerations for forward and backward compatibility in binary encoding
Forward compatibility means that you can have a new version of the schema as writer and an old version of the schema as reader.
Conversely, backward compatibility means that you can have a new version of the schema as reader and an old version as writer.
For detailed information, google [schema evolution](https://en.wikipedia.org/wiki/Schema_evolution)
**Field tags**

Forward compatibility - If, as the examples show, field tags are tied to each unique key, an introduction of a new key must be accompanied by the introduction of a new field tag. 

Backward compatibility - You can't add a new required field, except if you specify a default value. You can't remove fields except if they're optional.

**Datatypes**

Backward compatibility - Issues may arise moving from single-valued to multi-valued (eg. array) fields; Another example is converting numerical data - 32bit to 64bit. Issues and workarounds differ based on datatypes and the type of encoding used - for example, because Avro doesn't store the datatype anywhere except the schema, it may be easier to convert types there.

**Avro**
![Avro evolution](https://notes.shichao.io/dda/figure_4-6_600.png)

- It's no problem if the writer's schema and the reader's schema have their fields in a different order, because the schema resolution matches up the fields by field name.
- If the code reading the data encounters a field that appears in the writer's schema but not in the reader's schema, it is ignored.
- If the code reading the data expects some field, but the writer's schema does not contain a field of that name, it is filled in with a default value declared in the reader's schema.


## How do computers pass messages along?
Methods mentioned in the book:
- Databases
- APIs
- Message-brokers
