A Novel C Programming Error Handling Paradigm (1.1)
Published on December 31, 2019
Kaywan Joseph Mansubi
I stand and deliver where healthcare and technology intersect.

In this article I describe a novel way to concisely facilitate error handling in C.

Errors are normally returned as integer values in C. These often correspond to imported integer constants (or the notorious errno constants). My solution minimizes namespace pollution while optimizing conciseness, readability, and maintainability. The key is abstraction, namely to cast integers to bitfield types.

To begin, a bitfield is defined as a type:

// function_name.h
#pragma once
typedef struct function_name_err_t {
    // up to 32 fields for 64bit (bitfield structs are integer sized)
    unsigned unsupported_opcode: 1     ;
    unsigned close_frame       : 1     ;
    unsigned incomplete_frame  : 1     ;
    unsigned fatal_error       : 1     ;
} function_name_err            ;
Its eponymous function instantiates, manipulates, then returns the type, cast as an integer:

(Note: a more cohesive method of declaring an error as an integer and returning as an integer follows this illustrative introduction. Please see ensuing Note: for the embedded programmer.)

int function_name(/* ... */) {
  function_name_err ERR   ; // place a bitfield struct on the stack
  
  *(int *)&ERR = 0        ; // zero all bitfields
  // ...
  if (/*...*/) {
    ERR.close_frame=1     ;
    ERR.fatal_error=1     ;
  }
              
  if (ERR.fatal_error != 1) {
    // ... proceed
  }
  return *(int *)&ERR     ; // cast to int for return       
}
The calling function can optionally inspect the returned integer via type-casting:

#include "function_name.h"
int main() {
   // ...
   int result                            ;
   result = function_name(/* ... */)     ;
   if ( result!=0 ) {
      if ( (*(function_name_err*)&result).fatal_error ) {
         // printbits(result)       ;
         return 1                   ;
      } 
   }
As shown, a returned error can be handled by casting the integer to the imported bitfield type then accessing the named bits. Boolean combinations of named bits may also be tested. This approach keeps error handling in C simple with the option to add complexity.

Additionally, the bits can be printed as a stream of 32 1's and 0. This permits for concise, fast error interpretation at a glance. The first named field statement (in this case unsupported_opcode) corresponds to the least significant bit, so when printed the output would look like:

00000000000000000000000000001010

So, reading from right-to-left corresponding to top-to-bottom declarations, we have: unsupported_opcode=0, close_frame=1, incomplete_frame=0, fatal_error=1. Now it is immediately visible that there are two errors, which can be quickly identified by referencing the struct definition. To further illustrate, putting "fatal_error" at the top of the struct definition would make it appear as the lsb (rightmost bit), providing a meaningful visual impact. Note: because an integer is a multi-byte value, internal ordering may be reversed on a big-endian processor. Nevertheless, the boolean logic using named fields is endian agnostic.

Bitfield utilization outdoes the tradition of using an enumerated type or defining constants for a couple of reasons: one, it obviates bitwise operations when testing multiple conditions; and two, it brings to bear error-states via code-completion. Its appeal also encourages standards compliant coding, restricting signal trapping to its designated domain. On the other hand, in production, defining constants is the traditional approach, presumably to ensure compiler optimization (see proceeding note). For such projects, this approach might be better for debugging or private-facing error handling.

Note: for the embedded programmer, complexity could be reduced by declaring the error variable as an integer type to be returned directly without casting and casting only on error assignments. This may ensure consistent compiler/runtime optimizations impacted by bitfield struct instantiation. This would be a statement like:

// typedef function_name_err ...
int function_name(/* ... */) {
     int ERR = 0                                       ;
     if (/*...*/) {
          (*(function_name_err*)&ERR).fatal_error=1    ;
     }
     // ...
     return ERR                                        ;
}
Casting on assignment as opposed to declaring as the type has the added benefit of being able to reuse the integer in other contexts, such as receiving integer error values from other functions called. The primary difference between this way and the previous is the need to type the cast for each bitfield assignment, that is, assignment just requires more typing. Also, for re-used variables, it will be necessary to precede a new assignment with a zero assignment. This is my preferred way for using bitfields in this context, but the previous way helped illustrate the idea.

The technique may solve problems for C++ programmers, where there is temptation to throw exceptions instead of returning error numbers, which results in unnecessary overhead and a distortion of the Standard's design goals.

In closing, preferring integer returns with the option to easily abstract them not only results in more readable, compact code, but also encourages more standard compliant coding and consequently more optimized code. It is hoped that the reader has found my technique useful or in the least has been inspired by it.



Kaywan Joseph Mansubi

12/31/2019

I have taken a few courses in programming C and C++, none of which covered bitfields, which as I have demonstrated can be a very useful construct! If you consider yourself a beginner C programmer, the following book, which also describes bitfields, is recommended: Teach Yourself C by Herbert Schildt.

Disclaimer: This is an original article for which I have no outside citations. However, just as one can say every chord progression on a guitar has been played at some point, every C construct has appeared somewhere. Therefore, while the insights I have shared are wholly my own from having programmed extensively in C and C++ in my spare time, I doubt I am the first person to have employed this method. Finally, this technique has not been thoroughly tested but in the very least it should serve to reinforce C programming language concepts for the reader.
