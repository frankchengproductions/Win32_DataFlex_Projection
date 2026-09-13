# DataFlex Language Projection of Win32 MetaData

## Summary

The DataFlex language can consume most Win32 APIs. However the process of consuming those APIs requires manually looking up documentation online and translating all the related APIs / structs / enums / constants correctly from C++ to DataFlex. This project will do the grunt work for you. All you need is a multi-file search tool to find what you need. After that it's just copy/paste.

## Folder Structure
The folder structure of this project resembles the structure within the Win32 MetaData project. Within each folder, there will be 5 types of files.
* Win32Alias-##.pkg - Define `[One Data Type]` For `[Another Data Type]`
* Win32API-##.pkg - External_Function
* Win32Constant-##.pkg - Define `[Constant's Name]` For `[Literal Value]`
* Win32Enum-##.pkg - Enum_List/End_Enum_List
* Win32Struct-##.pkg - Struct/End_Struct

## Unions
The DataFlex language does not support Union data structure. In order to bring in the necessary structures from C++ to DataFlex, we have to make some concessions. Take a look at the API function [SendInput](https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-sendinput) and the parameter type [INPUT](https://learn.microsoft.com/windows/win32/api/winuser/ns-winuser-input). In DataFlex, they are defined as 

```dataflex
External_Function SendInput "SendInput" USER32.dll ;
	UInteger cInputs ;
	Pointer pInputs ;	// INPUT*
	Integer cbSize ;
	Returns UInteger

Struct _Anonymous_e__Union_1701
	// MOUSEINPUT mi // (32)
	// KEYBDINPUT ki // (24)
	// HARDWAREINPUT hi // (8)
	MOUSEINPUT mi
End_Struct

Struct INPUT
	UInteger type	// INPUT_TYPE
	Integer iMissingAlignment1
	_Anonymous_e__Union_1701 Anonymous
End_Struct
```
`_Anonymous_e__Union_1701` is an Union data type containing 3 members with different sizes. The number at the end of each member represents the size of the data structure. In C++, an Union will take on the size of its largest member. In this case, `MOUSEINPUT mi`. What if you need to use `KEYBDINPUT ki`? You will have to make the following modification

```dataflex
Struct _Anonymous_e__Union_1701
	// MOUSEINPUT mi // (32)
	// KEYBDINPUT ki // (24)
	// HARDWAREINPUT hi // (8)
	KEYBDINPUT ki
	UChar[8] uMissingAlignment
End_Struct
```
Since the largest member is `MOUSEINPUT`, which is 32 byte long. `KEYBDINPUT`, however, it's only 24 byte long. Therefore you have to make up the difference in size by padding the structure for 8 more bytes (32 bytes - 24 bytes). The `UChar[8] uMissingAlignment` is the padding. The commented out lines within `_Anonymous_e__Union_1701` are for future references so that you can make modifications easily without manually counting the size of each member.

## GUID
GUID is a common data type in Win32 C++. This is how you define a GUID in Win32 using C++.
```cpp
DEFINE_GUID(FOLDERID_Desktop, 0xB4BFCC3A, 
0xDB2C, 0x424C, 0xB0, 0x29, 0x7F, 0xE9, 0x9A, 0x87, 0xC6, 0x41);
```
In DataFlex, we define them as string.
```dataflex
Define FOLDERID_Desktop	For "b4bfcc3a-db2c-424c-b029-7fe99a87c641"
```
You will have to call [UuidFromString](https://learn.microsoft.com/windows/win32/api/rpcdce/nf-rpcdce-uuidfromstringa) to convert a string to a GUID. Here is a sample code

```dataflex
Use UI

Define FOLDERID_Desktop	For "b4bfcc3a-db2c-424c-b029-7fe99a87c641"

Struct Guid
	UInteger Data1
	UShort Data2
	UShort Data3
	UChar[8] Data4
End_Struct

External_Function UuidFromStringA "UuidFromStringA" RPCRT4.dll ;
	String StringUuid ;	// PSTR
	Pointer Uuid ;	// Guid*
	Returns Integer	// RPC_STATUS

Guid FOLDERID_Desktop_Guid
Integer iStatus
Move (UuidFromStringA( ;
  FOLDERID_Desktop, ;
  AddressOf(FOLDERID_Desktop_Guid))) ;
  to iStatus
```

## Remarks
* This DataFlex Language Projection translates complete data structures / constants / APIs / enumerations / data types.
* All structure alignments are specifically translated for running under 64 bit Windows (As all 32 bit Windows are out of support)
* If an API's parameter type is defined as `[in] PSTR` or `[in] PWSTR` in the Win32 MetaData project, it will be translated as `String` and `WString` correspondingly. `[in/out]` or `[out]` parameters are translated as `Pointer` because the API can potentially modify the content of the parameter.
* `Void_Type` (defined in FMAC) will be used for the return type if an API doesn't return any value.

## Acknowledgement
* Microsoft - for providing the metadata for the entire Win32 API system

## Important Links
* GitHub repository for the [Win32 MetaData project](https://github.com/microsoft/win32metadata)
* Download the actual [Win32 MetaData](https://www.nuget.org/packages/Microsoft.Windows.SDK.Win32Metadata/)
* This DataFlex Language Projection is based on [71.0.26 Preview](https://www.nuget.org/packages/Microsoft.Windows.SDK.Win32Metadata/71.0.26-preview) version of the Win32 MetaData project
