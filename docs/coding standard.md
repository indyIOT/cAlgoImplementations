# C Coding Standard

## File Header

Every file starts with this block:

```c
/**********************************************************************************
 * @file filename.h
 * @brief Brief description
 * @author Anthony Garza
 * @date YYYY-MM-DD (use current date if none given)
 * @details Detailed description line 1
 * @details This implementation is designed to work with the Raspberry Pi Pico and the C programming language.
 * @copyright Copyright (c) 2025 DeviceIQ. All rights reserved.
 */
```

## Include Guards / C++ Compatibility

Required order, every header:

```c
#ifndef HEADER_NAME_H
#define HEADER_NAME_H

#ifdef __cplusplus
extern "C" {
#endif

// Content here

#ifdef __cplusplus
}
#endif

#endif /* HEADER_NAME_H */
```

## Section Organization

Separate sections with:

```c
/*============================================================================
 * SECTION NAME
 *============================================================================*/
```

Header files, in order:
1. Includes
2. Constants and Definitions / Defines and Constants
3. Types and Structures / Enumeration Definitions
4. Public Function Prototypes (grouped by function)

Source files, in order:
1. Includes
2. Constants and Definitions / Defines and Constants
3. Types and Structures / Enumeration Definitions
4. Static Variables
5. Static Function Prototypes
6. Public Function Implementations (grouped by function)
7. Static Function Implementations (grouped by function)
8. Interrupt Function Implementations

## Formatting

Allman braces, always on their own line.

```c
if ( condition )
{
    // code
}

typedef struct
{
    int member;
} sStructName_t;
```

Spacing:
- space after control keywords (`if`, `while`, `for`, `switch`)
- space after `(` and before `)`
- spaces around binary operators (`==`, `!=`, `<`, `>`, `+`, `-`, `*`, `/`, `=`)
- no spaces inside `[]`

```c
if ( x == 0 )
while ( ( i < MAX ) && ( j > MIN ) )
function( param1, param2 )
array[index]
```

Indentation is spaces only, never tabs, 4 per level. No trailing whitespace on any line.

## Naming

Variables and functions are camelCase.

```c
int variableName;
void functionName( void );
```

Type prefixes:
- `s` for structs - `sStructName_t`
- `e` for enums - `eEnumName_t`
- `i` for integers - `iCustomInt_t`
- `p` for pointers - `pCustomPointer_t`

Acronyms stay consistent within a file - all uppercase or all lowercase, never mixed.

```c
// good
sDhcpServer_t, sDhcpMessage_t

// also good
sDHCPServer_t, sDHCPMessage_t

// bad, picked a lane and swerved out of it
sDHCPServer_t, sDhcpMessage_t
```

## Pointers

- `int * ptr` - space on both sides of the asterisk
- `int const * ptr` - const goes after the type
- `int const * const ptr` - both consts when both apply

## Magic Numbers

Nothing but `0` gets to be a bare literal. Everything else is a named constant.

```c
// good
#define MAX_BUFFER_SIZE     256
#define DEFAULT_TIMEOUT_MS  1000
if ( count < MAX_BUFFER_SIZE )

// bad
if ( count < 256 )
delay( 1000 );

// fine - 0/NULL doesn't need a name, everyone already knows what it means
if ( ptr == NULL )
for ( i = 0; i < max; i++ )
```

## Control Flow

No early exits. One return, at the bottom of the function.

```c
sErrorStruct_t function( void )
{
    sErrorStruct_t retValue = BLANK_ERROR_STRUCT;

    if ( condition )
    {
        retValue = CREATE_ERROR( ERROR_CODE );
    }
    else
    {
        // do work
    }

    return retValue;  // single exit point
}
```

## Documentation

Every function gets the same Doxygen block above it in both the header and the `.c` file:

```c
/**
 * @brief Brief description of function purpose
 *
 * @param paramName Description of parameter
 * @param data      Pointer to data buffer
 * @param length    Length of data in bytes
 *
 * @return sErrorStruct_t Error structure indicating result
 *
 * @note Additional important information
 * @warning Critical warnings about usage
 */
sErrorStruct_t functionName( type * const paramName, uint8_t const * const data, size_t const length );
```

The implementation gets the identical block, not a shortened version.

Comments are `/* */`, not `//`. Be verbose about it - write comments assuming whoever reads
this next is reading at a 4th grade level. Mostly a joke, not completely.

One blank line at the end of every file.

## A Few More Rules

- static variables are file scope, stack variables are function scope
- initialize variables where you declare them, unless it costs more flash than it's worth -
  memset instead in that case
- no magic numbers except 0, no tabs, no trailing whitespace, single return point, matching
  Doxygen above prototype and implementation - if you only remember one section, remember these
