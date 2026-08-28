# DataFlex Language Projection of Win32 MetaData

## Summary

The DataFlex language can consume most Win32 APIs. However the process of consuming those APIs requires manually looking up documentation online and translating all the related APIs / structs / enums / constants correctly from C++ to DataFlex. This project will do the grunt work for you. All you need is a multi-file search tool to find what you need. After that it's just copy/paste.

## Folder Structure
The folder structure of this project resembles the structure within the Win32 MetaData project. Within each folder, there will be 3 types of files.
* Win32Alias-##.pkg - Define `[One Data Type]` For `[Another Data Type]`
* Win32API-##.pkg - External_Function
* Win32Constant-##.pkg - Define `[Constant Name]` For `[Literal Value]`
* Win32Enum-##.pkg - Enum_List/End_Enum_List
* Win32Struct-##.pkg - Struct/End_Struct

## Remarks
* This DataFlex Language Projection only translates the necessary data structures / constants in order to call all the APIs included.
* DataFlex does not support union data type. Thus all unions are defined as `UChar[Length] Union#`
* All structure alignments are specifically translated for running under 64 bit Windows (As all 32 bit Windows are out of support)

## Acknowledgement
* Microsoft - for providing the metadata for the entire Win32 API system

## Important Links
* GitHub repository for the [Win32 MetaData project](https://github.com/microsoft/win32metadata)
* Download the actual [Win32 MetaData](https://www.nuget.org/packages/Microsoft.Windows.SDK.Win32Metadata/)
* This DataFlex Language Projection is based on [71.0.20 Preview](https://www.nuget.org/packages/Microsoft.Windows.SDK.Win32Metadata/71.0.20-preview) version of the Win32 MetaData project
