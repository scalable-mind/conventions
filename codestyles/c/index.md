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
- system → libs → user-defined;

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

> :information_source: As many text editors support `.editorconfig` files for maintaining consistency of codestyles, you can place one in the root directory of a project. You can download it [here](https://raw.githubusercontent.com/scalable-mind/public-assets/master/editorconfig/.editorconfig) by just pressing `Ctrl+S` ([about EditorConfig](http://editorconfig.org/)).

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

A *module* is a C structure containing a set of function pointers. The main goal of this structure is to provide the one entry point to functions considered as a single interface. Also, when using a pattern *dependency injection (or DI)*, you can manage several implementations of the same module, which makes the development more flexible and customizable.

Any module's definitioin consists of two parts: a **structure** that contains all the function pointers, and a special **function** that returns the same instance of the module. We will define and implement a simple module called `MyModule`.

The module's instance function should be designed as a *singleton*: it always returns the same instance of the module. The module instance should be stored in a static variable in its instance function's scope. The function also attaches the implementations of the module methods to the function pointers in the module instance verifying they are not `NULL`s. The methods implementations should be defined as `static`, so they will remain outside the global scope. So all implementations should be defined in the same file.

> :page_facing_up: `./my_module.h`:
```c
#ifndef MY_MODULE_H
#define MY_MODULE_H

typedef struct {

    void (*do_something)();

} MyModule;

MyModule my_module();

#endif
```

> :page_facing_up: `./my_module_impl.c`:
```c
#include "./my_module.h"

static void do_something() {    // is seen only inside my_module_impl.c
    // some operations
}

MyModule my_module() {
    static MyModule instance;
    if (instance.do_something == NULL) {
        instance.do_something = do_something;
    }
    return instance;
}
```

Now we can use `MyModule` like this:
```c
my_module().do_something();
```

Modules can be designed in several ways: as *utility modules* or as *API modules*. Moreover, the module that you are developing should be explicitly specified, whether it is that a utility or an API (see [Naming](#naming) for modules).

#### 1. Utility Modules
If the module contains some utility functions, then the module is considered as a kind of namespace. We'll call this *utility module*.

#### 2. API Modules
If the module is designed around some data structure ("class"), then the module is considered as an API for this particular data structure. We'll call this *API module*.

API modules may define and implement some methods for creating/deleting objects they designed for. These methods should be called as `init` and `del` respectively. We will define an API module called `MyDataApi` whose methods perform actions on the `MyData` structure.

> :page_facing_up: `./my_data.h`:
```c
#ifndef MY_DATA_H
#define MY_DATA_H

typedef struct {

    int id;

    char* name;

} MyData;

#endif
```

> :page_facing_up: `./my_data_api.h`:
```c
#ifndef MY_MODULE_H
#define MY_MODULE_H

#include "./my_data.h"

typedef struct {

    MyData* init(int id, const char* name);     // creates a new object

    void del(MyData* self);     // deletes an object

    void fill_filename(MyData* self, char filename[]);

    // ... other methods interacting with MyData objects

} MyDataApi;

MyDataApi my_data_api();

#endif
```

> :page_facing_up: `./my_data_api_impl.c`:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "./my_data_api.h"

static MyData* init(int id, const char* name) {
    MyData* my_data = (MyData*) malloc(sizeof(MyData));

    my_data->id = id;

    my_data->name = (char*) calloc(strlen(name), sizeof(char));
    strcpy(my_data->name, name);

    return my_data;
}

static void del(MyData* self) {
    free(self->name);
    free(self);
}

static void fill_filename(MyData* self, char filename[]) {
    sprintf(filename, "%d_%s.jpg", self->id, self->name);
}

MyDataApi my_data_api() {
    static MyDataApi instance;
    if (instance.init == NULL) {
        instance.init = init;
    }
    if (instance.del == NULL) {
        instance.del = del;
    }
    if (instance.fill_filename == NULL) {
        instance.fill_filename = fill_filename;
    }
    return instance;
}
```

> Usage:
```c
MyData* lora_palmer = my_data_api().init(1, "Lora Palmer");

// ...

char filename[256];
my_data_api().fill_filename(lora_palmer, filename);

// ...

my_data_api().del(lora_palmer);
```
