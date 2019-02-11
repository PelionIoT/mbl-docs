## Design and coding style

Be consistent with your changes. Some of the main principles are:

* OpenEmbedded layers and recipes: [https://www.openembedded.org/wiki/Styleguide](https://www.openembedded.org/wiki/Styleguide).
* Python code:
  * Mandatory PEPs (Python Enhancement Proposals): PEP20, PEP8, PEP257.
  * Automated check scripts in our `mbl-tools` repository: [https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.5/ci/sanity-check](https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.5/ci/sanity-check).


## C++ coding style guide

### Names

#### Global variables

* Global variables have names with a `g_` prefix (e.g. `g_time_prefix_format`).

#### Classes and Structs

* Names of classes, structs and aliases for them are in `UpperCamelCase`. When the name includes an initialism, only the first letter of the initialism is capitalized (e.g. `MblCloudClient`).

* Use `class` rather than `struct` whenever it contains anything but public data members.

* Names of data members are in `lower_snake_case`.

* Static data member names have an `s_` prefix (e.g. `s_instance`).

#### 2. Enums

* Names of enum types and aliases for them are in `UpperCamelCase`. When the name includes an initialism, only the first letter of the initialism is capitalized (e.g.  `MblError`).

* In C++98 code, enumerator names are usually prefixed with the name of the enum type and an underscore (e.g. `MblError_None`) to mitigate the lack of scoping for enumerator names. In C++11 code that doesn't touch C/C++98 interfaces (and isn't likely to in the future) then scoped enums should be used instead.

#### 3. Methods and Free Functions

* Names of methods and free functions are in `lower_snake_case` except when the name includes a type, when the type name is used verbatim (e.g. `MblError_to_str`).


#### Namespaces

* Namespace names are in `lower_snake_case`.

* Closing braces for namespace blocks are commented with the name of the namespace. For example:

  ```
  namespace mbl {

  /**
   * Initialize the log and trace mechanisms. After calling this the mbed-trace
   * library can be used for logging.
   */
  MblError log_init();

  } // namespace mbl
  ```

#### Files

* Files containing code for a single class are named after that class (e.g. `MblCloudClient.h` and `MblCloudClient.cpp`).

* Files containing code for a single function are named after that function.

* Files containing a number of functions are given a descriptive name in `lower_snake_case` such as `update_handlers.h` and `update_handlers.cpp`

### Whitespace

* Indent 4 spaces at a time (no tabs).


* Function/method definitions look like this:

    ```C++
    ReturnType ClassName::method_name(ParamType1 param1, ParamType2 param2)
    {
        // Stuff
    }
    ```

    or if that gets too long:

    ```C++
    ReturnType ClassName::method_name(
        ParamType1 param1,
        ParamType2 param2,
        ParamType3 param3,
        ParamType4 param4)
    {
        // Stuff
    }
    ```

    or if there's a constructor's initialization list:

    ```C++
    ClassName::ClassName(ParamType1 param1, ParamType2 param2)
        : member1_(param1)
        , member2_(param2)
    {
        // Stuff
    }
    ```

    or possibly:

    ```C++
    ClassName::ClassName(
        ParamType1 param1,
        ParamType2 param2,
        ParamType3 param3,
        ParamType4 param4
    )
        : member1_(param1)
        , member2_(param2)
    {
        // Stuff
    }
    ```

* C++98 enum definitions look like this:

    ```C++
    enum EnumType {
        EnumType_Value1,
        EnumType_Value2,
        EnumType_Value3,
    };
    ```

* Namespace blocks are not indented:

    ```C++
    namespace mbl {

    class MblCloudClient
    {
        // stuff
    };

    } // namespace mbl
    ```

* `if`s look like:

    ```C++
    if (condition) {
        // Stuff
    }
    else {
        // Other stuff
    }
    ```

* `for`s look like:

    ```C++
    for (i = 0; i < n; ++i) {
        // Stuff
    }
    ```

* `while`s look like:

    ```C++
    while (condition) {
        // Stuff
    }
    ```

* `switch`es look like:

    ```C++
    switch (value) {
        case EnumValue1: return stuff;
        case EnumValue2: return other_stuff;
    }
    ```

    or

    ```C++
    switch (value) {
        case EnumValue1:
            DoStuff();
            break;
        case EnumValue2:
            DoOtherStuff();
            break;
    }
    ```

### Other guidelines

mbl-cloud-client contains a mixture of C++98 and C++11, and in certain places must provide C-style APIs. Not all of the following guidelines will apply to both C++98 and C++11 and C-style APIs.

The list of C++ Core Guidelines below is not a complete list of the C++ Core Guidelines that should be followed in this project, it is merely the highlights.

* Don't use exceptions
* Switch statements over an enum value should not contain a `default` case so that the compiler can catch missing cases
* From the C++ Core Guidelines:
    * [I.2: Avoid non-`const` global variables](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i2-avoid-non-const-global-variables)
    * [I.3: Avoid singletons](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i3-avoid-singletons)
    * [I.22: Avoid complex initialization of global objects](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i22-avoid-complex-initialization-of-global-objects)
    * [C.4: Make a function a member only if it needs direct access to the representation of a class](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-member)
    * [C.5: Place helper functions in the same namespace as the class they support](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-helper)
    * [C.7: Don't define a class or enum and declare a variable of its type in the same statement](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c7-dont-define-a-class-or-enum-and-declare-a-variable-of-its-type-in-the-same-statement)
    * [C.9: Minimize exposure of members](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c9-minimize-exposure-of-members)
    * [C.10: Prefer concrete types over class hierarchies](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c10-prefer-concrete-types-over-class-hierarchies)
    * [C.20: If you can avoid defining default operations, do](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c20-if-you-can-avoid-defining-default-operations-do)
    * [C.21: If you define or =delete any default operation, define or =delete them all](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c21-if-you-define-or-delete-any-default-operation-define-or-delete-them-all)
    * [C.22: Make default operations consistent](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-zero)
    * [C.35: A base class destructor should be either public and virtual, or protected and nonvirtual](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-zero)
    * [C.46: By default, declare single-argument constructors explicit](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c46-by-default-declare-single-argument-constructors-explicit)
    * [C.47: Define and initialize member variables in the order of member declaration](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c47-define-and-initialize-member-variables-in-the-order-of-member-declaration)
    * [C.49: Prefer initialization to assignment in constructors](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c49-prefer-initialization-to-assignment-in-constructors)
    * [C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c128-virtual-functions-should-specify-exactly-one-of-virtual-override-or-final)
    * [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c129-when-designing-a-class-hierarchy-distinguish-between-implementation-inheritance-and-interface-inheritance)
    * [C.132: Don't make a function virtual without reason](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c132-dont-make-a-function-virtual-without-reason)
    * [C.133: Avoid protected data](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c133-avoid-protected-data)
    * [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to delete objects created using new](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c149-use-unique_ptr-or-shared_ptr-to-avoid-forgetting-to-delete-objects-created-using-new)
    * [C.164: Avoid conversion operators](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c164-avoid-conversion-operators)
    * [R.1: Manage resources automatically using resource handles and RAII (Resource Acquisition Is Initialization)](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r1-manage-resources-automatically-using-resource-handles-and-raii-resource-acquisition-is-initialization)
    * [R.3: A raw pointer (a `T*`) is non-owning](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r3-a-raw-pointer-a-t-is-non-owning)
    * [R.4: A raw reference (a `T&`) is non-owning](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r4-a-raw-reference-a-t-is-non-owning)
    * [R.10: Avoid `malloc()` and `free()`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r10-avoid-malloc-and-free)
    * [R.12: Immediately give the result of an explicit resource allocation to a manager object](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r12-immediately-give-the-result-of-an-explicit-resource-allocation-to-a-manager-object)
    * [R.20: Use `unique_ptr` or `shared_ptr` to represent ownership](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#rsmart-smart-pointers)
    * [R.30: Take smart pointers as parameters only to explicitly express lifetime semantics](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r30-take-smart-pointers-as-parameters-only-to-explicitly-express-lifetime-semantics)
    * [ES.1: Prefer the standard library to other libraries and to "handcrafted code"](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es1-prefer-the-standard-library-to-other-libraries-and-to-handcrafted-code)
    * [ES.5: Keep scopes small](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es5-keep-scopes-small)
    * [ES.10: Declare one name (only) per declaration](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es10-declare-one-name-only-per-declaration)
    * [ES.12: Do not reuse names in nested scopes](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es12-do-not-reuse-names-in-nested-scopes)
    * [ES.21: Don't introduce a variable (or constant) before you need to use it](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es21-dont-introduce-a-variable-or-constant-before-you-need-to-use-it)
    * [ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on)
    * [ES.26: Don't use a variable for two unrelated purposes](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es26-dont-use-a-variable-for-two-unrelated-purposes)
    * [ES.47: Use `nullptr` rather than `0` or `NULL`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es47-use-nullptr-rather-than-0-or-null)
    * [ES.49: If you must use a cast, use a named cast](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es49-if-you-must-use-a-cast-use-a-named-cast)
    * [ES.71: Prefer a range-for-statement to a for-statement when there is a choice](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es71-prefer-a-range-for-statement-to-a-for-statement-when-there-is-a-choice)
    * [ES.75: Avoid do-statements](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es75-avoid-do-statements)
    * [CP.20: Use RAII, never plain `lock()`/`unlock()`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#cp20-use-raii-never-plain-lockunlock)
    * [CP.22: Never call unknown code while holding a lock (e.g., a callback)](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#cp22-never-call-unknown-code-while-holding-a-lock-eg-a-callback)
    * [NL.18: Use C++-style declarator layout](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl18-use-c-style-declarator-layout)
    * [NL.20: Don't place two statements on the same line](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl20-dont-place-two-statements-on-the-same-line)
    * [NL.25: Don't use `void` as an argument type](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl25-dont-use-void-as-an-argument-type)
    * [NL.26: Use conventional `const` notation](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl26-use-conventional-const-notation)
