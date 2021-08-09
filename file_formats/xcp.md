# .xcp file format

The .xcp file format is a file format, that is used for storing variables like casiobasic programs or the algy2 login credentials on the calculator fx-cp400 (and others).

The xcp file has the following structure:

## general .xcp structure

| | Name | Length | Type | Description |
| --- | --- | --- | --- | --- |
|1 | VCP        | 10 byte | ascii     | the text "VCP.XDATA" including the 0x00 terminator |
|2 | longnumber |  8 byte | hex ascii | the number 0x5f4d4353 (which would represent the text "\_MCS") |
|3 | folname_len|  2 byte | hex ascii | length of the folder name + 1 (because 0x00 terminator is included) |
|4 | folname    |2-9 byte | ascii     | name of the folder including 0x00 terminator |
|5 | varname_len|  2 byte | hex ascii | length of the var name + 1 (because 0x00 terminator is included) |
|6 | varname    |2-9 byte | ascii     | name of the var including 0x00 terminator |
|7 | block_31   |  8 byte | hex ascii | the number "00000031" |
|8 | folname2   | 16 byte | ascii     | name of the folder, padded with 0xff |
|9 | varname2   | 16 byte | ascii     | name of the var, padded with 0xff |
|10| length     |  4 byte | binary big endian | The length of the data block |
|11| var_type   | 13 byte | ascii/binary     | the type of the var, padded with 0xff, see explanation further down (a program/text var has the type "GUQ") |
|12|length_ascii|  8 byte | hex ascii | the value of length again, but in ascii hex, small letters |
|13| data       | 'length' bytes | | the actual data, see explanation further down |
|14| checksum   |  2 byte | hex ascii | the checksum (take 0x00 and subtract all byes, except hex-ascii values, these are subtracted as hexadecimal numbers. |

## xcp var_type

The field 11 contains the type of the variable, this is usually 3 bytes, mostly (but not always) upper case ascii letters, padded with 10 0xff characters.

The 3rd characr is assumed to always be an upper case ascii 'Q'.

The 2nd character is a 'U' when the variable is not marked as 'Locked' or an 'L' if the variable is marked as lockced. Locked variables have a lock symbol in the variable manager 
and they can only be deleted after choosing the menu option 'Unlock' in the 'Edit' menu in the variable manager.

The first byte is assumed to indicate the type of the variable, here is a list of all the known file types:
| var_type\[0\] | variable type | Description |
| ---  | --- | --- |
| "G"  | "Program" | for example program/text variable, format described further down |
| "N"  | "Mem" | for example Algy2 file, for example the activation file (this variable contains '0x0a + user_name + 0x00 + password + 0x00') or an algy save file. |
| 0x04 | "String" | this contains only a string terminated with 0x00 in the block data |
| 0x12 | "Expression" | Numbers, I haven't looked at those yet... |

## program/text variables

At least for me, the program/text is the most useful type of variable, because it can contain arbritrary human readable data. You can use txt2xcp to convert .txt files into .xcp files with this type.

The var_type (field 11) contains "GUQ" followed by 10 bytes of 0xff.

The data block is structured like this:

|  | Name | Length | Type | Description |
| --- | --- | --- | --- | --- |
|13.1| text_len   |  4 byte | binary little endian | the length of the text + 3 |
|13.2| block_zero |  9 byte | binary | nine times 0x00 |
|13.3| text       | len byte |  | the ascii or binary data or text (can contain every character from 0x00 to 0xff) |
|13.4| eof        |  2 byte | binary | The file terminator "0x00 + 0xff" |
|13.5| padding    |0-3 bytes | binary | pad the data so 'length' is a multiple of 4 (pad with (3-((len+2)&~0x03)) bytes) |

The padding is required, different versions of calculator and computer software use different characters for padding, but everything is fine here. (peresonally, I would use 0x00)

## The checksum

The checksum is field 14 of the .xcp file, this is direclty after the data block and before the end of the file. 
To calculate the checksum, use an 8 bit unsigned number and start counting down from 0. For everything that is written to the .xcp file, the value has to be subtracted from this number.
When everything else is written to the file, this number will be the checksum and needs to be appended to the file in hex ascii, lower case.

For every field that has 'binary', 'ascii' or anything else in the column 'Type', you have to subtract the literal value of a byte from this checksum. For example if you write 'A' to the file, you have to
subtract 0x41 from the checksum. Writing a 0 to the file does not change the checksum. 

For every field with the type 'hex ascii', you have to subtract the literal value, not the ascii values. For example the block 7 has 8 bytes: '3030303030303331', 
this represents the ascii number 00000031, so you have to substract 0x00, 0x00, 0x00 and 0x31 (instead of 0x30 0x30 0x30 ...)

