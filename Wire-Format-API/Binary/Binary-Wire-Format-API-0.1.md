# Goals

A compact encoding that both encodes the length of the fields and the type of the fields.

# Encoding Byte

For short fields, the first byte denotes the type of the field, for small numbers the value is encoded in the later 4 bits. For strings and fields the later 4 bits denote the length of the preceeding data. For large numerics or string/fields the special type is used see section below :

| comment                               | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| top three bits denotes a three Type   | x   | x   | x   |  x  |     |     |     |    |
| remaining bits denotes the size/value |     |     |     |     | x   | x  |  x   | x  |

## Types ( for small values )

| type   | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| Num (0-15)                            | 0   | 0   | 0   | 0   |     |     |     |    | 
| Num (16-31)                           | 0   | 0   | 0   | 1   |     |     |     |    | 
| Num (32-47)                           | 0   | 0   | 1   | 0   |     |     |     |    | 
| Num (48-63)                           | 0   | 0   | 1   | 1   |     |     |     |    | 
| Num (64-79)                           | 0   | 1   | 0   | 0   |     |     |     |    | 
| Num (80-95)                           | 0   | 1   | 0   | 1   |     |     |     |    | 
| Num (96-111)                          | 0   | 1   | 1   | 0   |     |     |     |    | 
| Num (112-127)                         | 0   | 1   | 1   | 1   |     |     |     |    | 
| Control Message                       | 1   | 0   | 0   | 0   |     |     |     |    | 
| Decimal                               | 1   | 0   | 0   | 1   |     |     |     |    | 
| Integer                               | 1   | 0   | 1   | 0   |     |     |     |    | 
| Special                               | 1   | 0   | 1   | 1   |     |     |     |    | 
| Field (0-15)                          | 1   | 1   | 0   | 0   |     |     |     |    |
| Field (16-31)                         | 1   | 1   | 0   | 1   |     |     |     |    |
| String (0-15)                         | 1   | 1   | 1   | 0   |     |     |     |    |
| String (16-31)                        | 1   | 1   | 1   | 1   |     |     |     |    | 

### Definition of a field and a string

In the following uri

```yaml
csp: //a-uri?view=map
```

'csp' is the Field. 

And '//a-uri?view=map' is the String 

## Types ( for larger values )

the types use the Control Messages

| Control message                       | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| ( all higher bits are the same )                   | 1   | 0   | 0   | 0   |     |     |     |    | 
| (0x82) - Nested Block 32bit len 82    | 1   | 0   | 0   | 0   | 0   | 0   | 1   |  0 | 
| (0x8A) - byte[]                       | 1   | 0   | 0   | 0   | 1   | 0   | 1   |  0 | 
| (0x8D) - long[]                       | 1   | 0   | 0   | 0   | 1   | 1   | 0   |  1 | 
| (0x8E) - paddding with 32bit length   | 1   | 0   | 0   | 0   | 1   | 1   | 1   |  0 | 
| (0x8E) - single padded byte           | 1   | 0   | 0   | 0   | 1   | 1   | 1   |  1 | 

## Decimal

| Decimal message                       | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| ( all higher bits are the same )                    | 1   | 0   | 0   | 1   |     |     |     |    | 
| (0x90) - 32bit floating point         | 1   | 0   | 0   | 1   | 0   | 0   | 0   |  0 | 
| (0x91) - 64bit floating point         | 1   | 0   | 0   | 1   | 0   | 0   | 0   |  1 | 

## Integer

| Integer message                      | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| ( all higher bits are the same )                    | 1   | 0   | 1   | 0   |     |     |     |    | 
| (0xA0) - 128bit uuid                  | 1   | 0   | 1   | 0   | 0   | 0   | 0   |  0 | 
| (0xA1) - unsigned 8bit int            | 1   | 0   | 1   | 0   | 0   | 0   | 0   |  1 |
| (0xA2) - unsigned 16bit int           | 1   | 0   | 1   | 0   | 0   | 0   | 1   |  0 |
| (0xA3) - unsigned 32bit int           | 1   | 0   | 1   | 0   | 0   | 0   | 1   |  1 |
| (0xA4) - signed 8bit int              | 1   | 0   | 1   | 0   | 0   | 1   | 0   |  0 |
| (0xA5) - signed 16bit int             | 1   | 0   | 1   | 0   | 0   | 1   | 0   |  1 |
| (0xA6) - signed 32bit int             | 1   | 0   | 1   | 0   | 0   | 1   | 1   |  0 |
| (0xA7) - signed 64bit int             | 1   | 0   | 1   | 0   | 0   | 1   | 1   |  1 |

## Special

Note for\<string\> the string encode by default is a stop bit encoded len folloed by a ISO-8851-9 string, see more on this at https://github.com/OpenHFT/RFC/blob/master/Stop-Bit-Encoding/

| Special message                             | 7bit   | 6bit   | 5bit   | 4bit   | 3bit   | 2bit   | 1bit   | 0bit  |
| ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---|
| ( all higher bits are the same )            | 1   | 0   | 1   | 1   |     |     |     |    | 
| (0xB0) - FALSE                              | 1   | 0   | 1   | 1   | 0   | 0   | 0   |  0 | 
| (0xB1) - TRUE                               | 1   | 0   | 1   | 1   | 0   | 0   | 0   |  1 |
| (0xB2) - time UTC (long)                    | 1   | 0   | 1   | 1   | 0   | 0   | 1   |  0 |
| (0xB3) - Date (joda UTF8-Str)               | 1   | 0   | 1   | 1   | 0   | 0   | 1   |  1 |
| (0xB4) - DateTime (joda UTF8-Str)           | 1   | 0   | 1   | 1   | 0   | 1   | 0   |  0 |
| (0xB5) - ZonedDateTime (joda  \<string\>)   | 1   | 0   | 1   | 1   | 0   | 1   | 0   |  1 |
| (0xB6) - type ( \<type\> +  \<string\>)     | 1   | 0   | 1   | 1   | 0   | 1   | 1   |  0 |
| (0xB7) - field (\<field\> +  \<string\>)    | 1   | 0   | 1   | 1   | 0   | 1   | 1   |  1 |
| (0xB8) - string (\<string\> +   \<string\>) | 1   | 0   | 1   | 1   | 1   | 0   | 0   |  1 |
| (0xB9) - eventname (\<eventname\> +  \<string\>)        | 1   | 0   | 1   | 1   | 1   | 0   | 1   |  0 |
| (0xBA) - fieldNumber (\<fieldNumber\> + stopbit encoded) | 1   | 0   | 1   | 1   | 1   | 0   | 1   |  1 |
| (0xBB) - NULL                               | 1   | 0   | 1   | 1   | 1   | 1   | 0   |  0 |
| (0xBC) - Type                               | 1   | 0   | 1   | 1   | 1   | 1   | 0   |  0 |

## Type
0xBC <prefix len ecoded string, len is encoded using stop bits>  

## Sequences, Maps and Marshables

the sequence area encoded using 

```
0x82 <four byte unsigned len, this is the length in bytes of the encoded block of preceeing data>
```

so if we were going to encode these simple 4 entries ( shown below in text yaml )

``` yaml
[a,b,c,de] 
``` 

they as binary wire this would encode to 

```
0x82 0x09 0x00 0x00 0x00 0xE1 0x61 0xE1 0x62 0xE1 0x63 0xE2 0x64 0x65
```
| byte | description |
| ---- | ------------------- |
| 0x82 | denoting a nested structure  |
| 0x09 0x00 0x00 0x00 | the number of bytes of data to follow ( in little endian ) |
| 0xE1 | next element is a string of len 1 |
| 0x61 | 'a' |
| 0xE1 | next element is a string of len 1 |
| 0x62 | 'b' |
| 0xE1 | next element is a string of len 1 |
| 0x63 | 'c' |
| 0xE2 | next element is a string of len 2 |
| 0x64 0x65 | 'de' |

NOTE: althought YAML teats sequences or map diffently for binary wire we ue simualar encoding, its just they will hold diffent information

```yaml
{f1: a, f2: de} 
``` 

they as binary wire this would encode to 

```
0x82 0x0B 0x00 0x00 0x00 0xC2 0x66 0x31 0xE1 0x62 0xC2 0x66 0x32 0xE2 0x64 0x65
```
| byte | description |
| ---- | ------------------- |
| 0x82 | denoting a nested structure |
| 0x0B 0x00 0x00 0x00 | the number of bytes of data to follow ( in little endian ) |
| 0xC2 |  next element is a feild of len 2 |
| 0x66 0x31 | 'f1' |
| 0xE1 | next element is a string of len 1 |
| 0x62 | 'b' |
| 0xC2 | next element is a feild of len 2 |
| 0x66 0x32 | 'f2' |
| 0xE2 | next element is a string of len 2 |
| 0x64 0x65 | 'de' |

# Example

using this encoding descibed above, the following YAML 
``` yaml
--- !!meta-data
csp: //path/service
tid: 123456789
--- !!data
entrySet: [
    {
    key: key-1,
    value: value-1
},
    {
    key: key-2,
    value: value-2
}
]
``` 

When encoded with BinaryWire would appear as:
```
00000000 1C 00 00 40 C3 63 73 70  EE 2F 2F 70 61 74 68 2F ···@·csp ·//path/
00000010 73 65 72 76 69 63 65 C3  74 69 64 A3 15 CD 5B 07 service· tid···[·
00000020 48 00 00 00 C8 65 6E 74  72 79 53 65 74 82 3A 00 H····ent rySet·:·
00000030 00 00 82 18 00 00 00 C3  6B 65 79 E5 6B 65 79 2D ········ key·key-
00000040 31 C5 76 61 6C 75 65 E7  76 61 6C 75 65 2D 31 82 1·value· value-1·
00000050 18 00 00 00 C3 6B 65 79  E5 6B 65 79 2D 32 C5 76 ·····key ·key-2·v
00000060 61 6C 75 65 E7 76 61 6C  75 65 2D 32             alue·val ue-2    
```

this is the java code that created this this binary output above 

``` java
@Test
public void testSequence() {
    Wire wire = createWire();
    writeMessage(wire);
    wire.flip();
    System.out.println(wire.bytes().toHexString());

    Wire twire = new TextWire(Bytes.elasticByteBuffer());
    writeMessage(twire);
    twire.flip();
    System.out.println(Wires.fromSizePrefixedBlobs(twire.bytes()));
}

private void writeMessage(Wire wire) {
    wire.writeDocument(true, w -> w
            .write(() -> "csp").text("//path/service")
            .write(() -> "tid").int64(123456789));
    wire.writeDocument(false, w -> w
            .write(() -> "entrySet").sequence(s -> {
                s.marshallable(m -> m
                        .write(() -> "key").text("key-1")
                        .write(() -> "value").text("value-1"));
                s.marshallable(m -> m
                        .write(() -> "key").text("key-2")
                        .write(() -> "value").text("value-2"));
            }));
}
```

# Layout

The expected format is

4 bytes length with the meta-data bit set.
3-byte field + "csp"
N-byte string + string
3-bytes field + "tid"
integer value as a int64.

In the future we will make the tid to be smaller. possibly just a byte or 3 (uint16).

Q.Where would the nested be involved? 

A.You might consider the whole document with its length a "nested" structure.

