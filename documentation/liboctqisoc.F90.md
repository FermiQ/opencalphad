# Overview

The file `liboctqisoc.F90` serves as a C interoperability layer for the `liboctq` Fortran module. It utilizes the Fortran `ISO_C_BINDING` feature to expose the functionalities of `liboctq` (a TQ interface for OpenCalphad) to applications written in C. This allows C programs to call Fortran subroutines and functions for thermodynamic calculations, database interaction, and results retrieval.

The module is structured into two main parts:
1.  A helper module `cstr` for converting strings between Fortran and C representations (handling null termination, etc.).
2.  The main module `liboctqisoc` which contains the C-bound wrapper functions.

# Key Components

## String Conversion (`cstr` module)

*   **`c_to_f_string(s) result(str)` (Function)**
    *   **Description:** Converts a C string (null-terminated character array) to a Fortran allocatable string.
    *   **Parameters/Arguments:** `s` (C character array, IN).
    *   **Returns:** `str` (Fortran character string, allocatable).

*   **`c_to_f_str(s, sty)` (Subroutine)**
    *   **Description:** Converts a C string to a fixed-length Fortran character string (length 24).
    *   **Parameters/Arguments:** `s` (C character array, IN), `sty` (Fortran character string len=24, OUT).
    *   **Returns:** None.

*   **`f_to_c_string(fstring, cstr)` (Subroutine)**
    *   **Description:** Converts a Fortran string (fixed length 24) to a C string (null-terminated character array).
    *   **Parameters/Arguments:** `fstring` (Fortran character string len=24, IN), `cstr` (C character array, OUT).
    *   **Returns:** None.

## C-Bound TQ Interface (`liboctqisoc` module)

This module provides C bindings for most of the subroutines found in `liboctq`. The naming convention is generally `c_` prepended to the original Fortran routine name.

**Examples of Key Wrapped Subroutines:**

*   **`c_tqini(n, c_ceq)`**
    *   **Description:** C binding for `tqini`. Initializes the TQ interface and provides a C pointer to the equilibrium data structure.
    *   **Parameters/Arguments:** `n` (Integer(c_int), IN), `c_ceq` (Type(c_ptr), OUT: C pointer to Fortran `gtp_equilibrium_data`).
    *   **Returns:** None.

*   **`c_tqrfil(filename, c_ceq)`**
    *   **Description:** C binding for `tqrfil`. Reads a thermodynamic database file.
    *   **Parameters/Arguments:** `filename` (C character array, IN), `c_ceq` (Type(c_ptr), INOUT: C pointer to equilibrium data).
    *   **Returns:** None.

*   **`c_tqrpfil(filename, nel, c_selel, c_ceq)`**
    *   **Description:** C binding for `tqrpfil`. Reads a database file with selected elements.
    *   **Parameters/Arguments:** `filename` (C char, IN), `nel` (Integer(c_int), IN), `c_selel` (Array of C pointers to C strings, IN), `c_ceq` (Type(c_ptr), INOUT).
    *   **Returns:** None.

*   **`c_tqsetc(statvar, n1, n2, mvalue, cnum, c_ceq)`**
    *   **Description:** C binding for `tqsetc`. Sets a calculation condition.
    *   **Parameters/Arguments:** `statvar` (C string, IN), `n1`, `n2` (Integer(c_int), IN), `mvalue` (Real(c_double), IN), `cnum` (Integer(c_int), OUT), `c_ceq` (Type(c_ptr), IN).
    *   **Returns:** None.

*   **`c_tqcalc(c_ceq, mode)`**
    *   **Description:** Calculates equilibrium with different methods.
    *   **Parameters/Arguments:** `c_ceq` (Type(c_ptr), INOUT), `mode` (Integer(c_int), IN: 0 for no grid minimizer, 1 for global grid minimization, 2 for careful default).
    *   **Returns:** None. Updates `c_niter`.

*   **`c_tqce(mtarget, n1, n2, mvalue, c_ceq)`**
    *   **Description:** C binding for `tqce`. Calculates equilibrium, potentially with a target.
    *   **Parameters/Arguments:** `mtarget` (C string, INOUT), `n1`, `n2` (Integer(c_int), IN), `mvalue` (Real(c_double), INOUT), `c_ceq` (Type(c_ptr), INOUT).
    *   **Returns:** None. Updates `c_ntup`, `c_niter`.

*   **`c_tqgetv(statvar, n1, n2, n3, values, c_ceq)`**
    *   **Description:** C binding for `tqgetv`. Retrieves calculation results.
    *   **Parameters/Arguments:** `statvar` (C string, IN), `n1`, `n2` (Integer(c_int), IN), `n3` (Integer(c_int), INOUT), `values` (Real(c_double) array, INOUT), `c_ceq` (Type(c_ptr), INOUT).
    *   **Returns:** None.

*   **`c_tqgcom`, `c_tqgnp`, `c_tqgpn`, `c_tqgpi`, `c_tqgpi2`, `c_tqgpcn2`, `c_tqgpcs`, `c_tqphsts`, `c_tqcph1`, `c_tqcph3`, `c_tqlr`, etc.**
    *   **Description:** Numerous other C bindings for information retrieval, status setting, phase property calculations, and debugging output, corresponding to their `liboctq` counterparts.

*   **Specialized C-Bound Functions:**
    *   **`c_noofcs(iph)`:** Gets number of composition sets for a phase.
    *   **`c_noconst(iph, ics, c_ceq)`:** Gets number of constituents in a phase.
    *   **`examine_gtp_equilibrium_data(c_ceq)`:** Prints contents of the Fortran `gtp_equilibrium_data` structure for debugging.
    *   **`get_stoichiometric_coef(...)` / `c_get_stoichiometry(...)`:** Gets stoichiometric coefficient.
    *   **`change_stoichiometric(...)` / `c_change_stoichiometric(...)`:** Modifies stoichiometric coefficient.
    *   **`c_gtpsave(c_filename, c_specification)`:** Saves OC environment.
    *   **`c_gtpread(c_filename, c_specification)`:** Reads OC environment.
    *   **`c_errors_number()`:** Returns the current error code from the underlying Fortran library.
    *   **`c_reset_errors_number()`:** Resets the error code.

# Important Variables/Constants

The `liboctqisoc` module exposes several variables with C binding for direct access or information:

*   **`c_niter` (Integer(c_int), bind(c)):** Stores the number of iterations from the last equilibrium calculation.
*   **`c_nel` (Integer(c_int), bind(c)):** Number of elements in the system.
*   **`c_maxc`, `c_maxp` (Integer(c_int), bind(c)):** Maximum number of components and phases, mirroring `maxc` and `maxp` from `liboctq`.
*   **`c_cnam(maxc)` (Type(c_ptr) array, bind(c)):** Array of C pointers, intended to point to C strings representing component names. These are linked to the Fortran `cnames` target array.
*   **`cnames(maxc)` (Character(len=25) array, target):** Fortran character array that holds component names, made accessible via `c_cnam`.
*   **`c_mass(maxc)` (Real(c_double) array, bind(c)):** Array for component masses.
*   **`c_ntup` (Integer(c_int), bind(c)):** Number of phase tuples in the system.
*   **`c_gtp_equilibrium_data` (TYPE, bind(c)):** A Fortran derived type with C binding, designed to be compatible with a C struct. It mirrors the fields of the Fortran `gtp_equilibrium_data` type from `liboceqplus`/`liboctq`, allowing the main equilibrium data structure to be passed between C and Fortran. This includes fields like `status`, `eqname`, `tpval`, pointers to conditions, phase results, etc.

# Usage Examples

This module is not intended to be used directly from Fortran. Its primary purpose is to be called from C code. A C application would include a header file declaring these C-bound Fortran functions and then call them as if they were native C functions.

```c
// Conceptual C usage example (header file not shown)
// #include "liboctq_iso_c.h" // Assume a header exists

// int main() {
//     void* c_eq_ptr = NULL; // Using void* for c_ptr
//     int n = 0;
//     int error_code;
//
//     c_tqini(&n, &c_eq_ptr); // Note: passing address of pointer
//     if (!c_eq_ptr) { /* handle error */ }
//
//     char filename[] = "mydatabase.tdb";
//     c_tqrfil(filename, c_eq_ptr);
//     error_code = c_errors_number();
//     if (error_code != 0) { /* handle error */ }
//
//     // ... set conditions using c_tqsetc ...
//     // ... calculate equilibrium using c_tqce or c_tqcalc ...
//     // ... retrieve results using c_tqgetv ...
//
//     c_tqfree(); // Clean up Fortran allocations
//
//     return 0;
// }
```
The Fortran code uses `type(c_ptr)` for opaque pointers to Fortran objects (like `gtp_equilibrium_data`) and `c_loc` to get their C addresses. String handling requires careful conversion between Fortran's fixed-length or allocatable strings and C's null-terminated strings, which is the role of the `cstr` module.

# Dependencies and Interactions

*   **`iso_c_binding` (Fortran Module):** This standard Fortran module is fundamental for all C interoperability, providing types like `c_char`, `c_int`, `c_double`, `c_ptr`, and the `bind(c)` attribute.
*   **`cstr` (Internal Module):** Provides essential string conversion utilities (`c_to_f_string`, `f_to_c_string`, etc.) for passing string data between C and Fortran.
*   **`liboctq` (Fortran Module):** This is the core dependency. `liboctqisoc` is essentially a wrapper around `liboctq`. Most of the `c_*` subroutines in `liboctqisoc` make calls to the corresponding original Fortran routines in `liboctq` (e.g., `c_tqini` calls `tqini`, `c_tqgetv` calls `tqgetv`).
*   **`gtp_equilibrium_data` (Fortran Type):** The structure of this critical data type (defined in `liboceqplus` and used by `liboctq`) is mirrored in `c_gtp_equilibrium_data` using C binding to allow it to be passed and partially interpreted by C code (though direct field access from C to the Fortran type requires careful handling).
*   **Error Handling:** Error codes (e.g., `gx%bmperr` from the underlying Fortran library) are exposed via `c_errors_number()` and can be reset by `c_reset_errors_number()`.
*   **Global Data:** Some routines interact with global data or status flags within the Fortran common blocks or modules (e.g., `c_set_grid_density`, `c_set_status_globaldata`).
