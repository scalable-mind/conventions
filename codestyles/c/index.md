# C Coding Style Convention

## Code

**End of file**: with trailing newline

**Indent**: 4 spaces

**Opening curvy braces**: at the same line with the statement, e.g.
``` C
if (...) {
    // some code
}
```
**Complex conditions**:
``` C
if (
    (foo == bar) // clarifying parenthesis in logic operators
    && (leet != speak)
    || false  
) {
    // code
}
```

**Single line comments**: with a space after `//`:
``` C
// this is a single line comment
```

**Structure fields** are surrounded with new lines:
``` C
struct MyStruct {

    int field;

    char[4] four_chr_str;

    // comments are surrounded, too

};
```

**`#include`** statements order:
- alphabetic;
- system `->` libs `->` user-defined;

Example:
``` C
#include <stdio.h>
#include <stdlib.h>

#include <GLFW/glfw.h> // for OpenGL
#include <SomeLib.h>

#include "./AUserHeader.h"
#include "../MyHeader.h"

// code starts here
```

> :information_source: As many of text editors support `.editorconfig` files for maintaining consistency of codestyles, you can place one in the root directory of a project. You can download it [here](https://raw.githubusercontent.com/scalable-mind/public-assets/master/editorconfig/.editorconfig) just pressed `Ctrl+S` ([about EditorConfig](http://editorconfig.org/)).

## Naming

### Variables

A variable name should explain the data it stores, each word is separated with an underscore `_`. The variable name is written in **lowercase**, e.g. `my_var`.

### Functions

Function names should reflect their actual purposes, contain a verb, and each word is separated with an underscore `_`. The function name is written in **lowercase**, e.g. `do_something`.

### Modules

All module functions respect [previous rule](#functions). The module names are written in **CamelCase**.

A utility module name should end with the -`Utils` suffix:
``` C
typedef struct {

    // some utils

} MyUtils;
```
An API module name should end with the -`Api` suffix and contain the name of the data structure that API was designed for:
``` C
typedef struct {

    // some fields

} MyData;

typedef struct {

    // some APIs for MyData

} MyDataApi;
```

## Common Design Patterns

### File Hierarchy

### Modules

A module can be designed in several ways:
1. if the module contains some utility functions, then the module is considered as a kind of namespace. We'll call this *utility module*. *Utility modules* do not use variables to hold their instance; instead, they are used as the following:
``` C
my_utils().do_something();
```
To define a Utility module, use this header template:

``` C
//header.h

typedef struct {

    void (*do_something)();

} MyUtils;

MyUtils my_utils();
```

And for implementation, you can write this:
``` C
//implementation.c

#include "./header.h"

static void do_something() {
    // some operations
}

MyUtils my_utils() {
    static MyUtils instance;
    if (instance.do_something == NULL) {
        instance.do_something = do_something;
    }
    return instance;
}
```

2. if the module is designed around some data structure ("class"), then the module is considered as an API for this particular data structure. We'll call this *API module*;
``` C
MyData* data = my_data_api().init();
```
3. ... other approaches may be described here.
