# A Left To Right Declaration Syntax For C++

## Introduction

This proposal presents a library component that introduces simple left-to-right (LTR) declaration syntax to C++.

C's original 'declaration mimics use' (DMU) concept focuses on expressions and has an appealing simplicity and elegance. The concept was consistently applied and avoided redundant syntax for declaring and using entities. However, it proved not to be universally practical and alternatives for some cases were introduced and became best practices. For example, C89 offered a more type-explicit alternative syntax for function declarations:

    int f(x,y); // K&R style (DMU)
    int f(int,int); // Alternative, current practice

C++ places emphasis on types rather than on expressions [1] and introduces additional concepts that do not work well with C's DMU syntax. For example, references have no associated 'use' syntax but must have distinct 'declaration' syntax, and member-pointers for which a DMU syntax would face the same problems as the original K&R function declarations.

I propose a library component that uses C++11 template type aliases to introduce a simple LTR declaration syntax that is more in line with C++'s emphasis on types, is more consistent with the typical C++ user's conception of declarations and with declarations for other templated types, and solves some issues currently dealt with using coding style conventions or mental tricks.

## Motivation

- Many find the DMU syntax difficult to read or write and current practices include methods and tricks to ease the burden. The methods typically aim to keep individual declarations simple and as close to a LTR style as possible. These include breaking complex compound types up using typedefs which can then be used as simple type-specifiers, spacing rules that place type modifiers to the 'left', and rules against declaring more than one variable per declaration in avoid breaking the illusion created by such spacing rules.

 Tricks for reading complex compound declarations include the 'right to left' rule [2] for relatively simple declarations. For more complex declarations there is the 'spiral rule'[3]. Additionally there are tools for deciphering declarations, such as cdecl.org's "C gibberish ↔ English" translator which can convert between C's syntax and LTR English descriptions.

 Although Read/writability of the existing syntax can be a problem for any user it is a particular problem for novice users unfamiliar with the underlying concept, and methods and tricks for dealing with it. 

- Compound types have a natural order in which layers of more complex types are built from simpler types. The 'spiral rule' amply demonstrates that this order does not share a direct relation with the natural order in which C++ is read and written.

- DMU syntax is necessarily inconsistent with the syntax for templated types. Templated types naturally use a consistent LTR syntax instead of the 'inside-out' or 'spiral' ordering of the built-in type modifiers.

- Language linkage is part of function types the syntax for declaring linkages but is not consistent with other type modifying syntax. Typedefs are required to manipulate a function type's language linkage below the level of a whole declaration.

- Projects and individuals can use the solution I propose here, however including a solution in the standard offers additional benefits such as easy, direct availability to novice users and potentially wider usage.

## Solution

    // header ltr_types
    
    namespace std {
    namespace ltr {
    
    template<typename T>                      using ptr       = T*;
    template<typename T>                      using ref       = T&;
    template<typename T>                      using rref      = T&&;
    template<typename T>                      using constant  = T const;
    template<typename T>                      using volat     = T volatile;
    template<typename T, class C>             using mem_ptr   = T C::*;
    template<std::intmax_t N, typename T>     using raw_array = T[N];
    template<typename RetT, typename... Args> using func      = RetT(Args...);
     
    extern "C" template<typename T>   using c_linkage   = T;
    extern "C++" template<typename T> using cpp_linkage = T;
    
    } // namespace ltr
    } // namespace std
 

With this library, declarations can be read and written in an intuitive LTR fashion.

- Manual typedefs are replaced by these reusable template type aliases.
- The LTR ordering means the conceptual order of compound types matches the order for reading and writing C++, making reading and writing easier and eliminating the need for tricks like the spiral and right-to-left rules.
- Limiting declarations to single variables is no longer necessary.
- The emphasis on types over expressions is more 'C++ like.'
- The template type alias syntax is consistent with the syntax for using other templated types.
- Language linkage type modifiers are consistent with other type modifiers and templates.

Additionally this library mixes well with the built-in syntax when desired. E.g:

    ptr<int> foo(); // vs. func<ptr<int>> foo;

Complex compound type example:

    // A declaration randomly generated by cdecl.org 
    // char * const (*(* const bar)[5])(int ); 

    constant<ptr<raw_array<5,ptr<func<constant<ptr<char>>,int>>>>> bar;

The LTR type resembles cdecl.org's 'English' explanation:

> declare bar as const pointer to array 5 of pointer to function (int) returning const pointer to char

## Disadvantages

- Using these template type aliases is not as compact as the built-in syntax.

- DMU syntax allows a single declaration to declare variables of different, though related, types.

- The LTR syntax is not universally usable. For example it cannot be used for a function definition or on a class definition:

        func<int,int> times_two 
        {
            return arg1*2;
        }

        ptr<struct S {}> sPtr;

- The use of template type aliases doesn't mix with auto:

        rref<auto> x = foo(); // auto &&x = foo();

        int bar(rref<auto> x); // int bar(auto &&x);

[1]: http://www.stroustrup.com/bs_faq2.html#whitespace
[2]: http://ieng9.ucsd.edu/~cs30x/rt_lt.rule.html
[3]: http://c-faq.com/decl/spiral.anderson.html
