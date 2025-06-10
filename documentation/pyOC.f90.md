# Overview

The Fortran module file `pyOC.f90` (module `RawOpenCalphad`) is specifically designed to create Python bindings for the OpenCalphad thermodynamic library, particularly the `liboctq` TQ (Thermo-Calc Query) interface. It acts as a wrapper layer that is processed by the `f2py` tool. `f2py` uses this Fortran source to generate a Python extension module, enabling Python programs to call these Fortran subroutines and functions to perform thermodynamic calculations and data retrieval.

The key strategy employed is to wrap the original Fortran routines from `liboctq` with a `py` prefix and to use intermediate derived types (`eq_wrapper`, `comp_wrapper`) to manage complex data structures (like Fortran pointers or arrays of strings) in a way that `f2py` can effectively translate for Python.

# Key Components

The module `RawOpenCalphad` defines several wrapper subroutines and two helper derived types. Most subroutines are direct calls to their counterparts in the `liboctq` module.

## Helper Derived Types

*   **`eq_wrapper`**
    *   **Description:** A Fortran derived type designed to encapsulate the main equilibrium data structure pointer from `liboctq`.
    *   **Fields:** `type(gtp_equilibrium_data), pointer :: ceq`
    *   **Purpose:** Allows `f2py` to manage the opaque Fortran pointer (`ceq`) that represents the state of an equilibrium calculation. Python code will interact with an instance of `eq_wrapper` as an f2py-generated object.

*   **`comp_wrapper`**
    *   **Description:** A Fortran derived type to bundle component information (number of components and their names).
    *   **Fields:** `integer :: n`, `character(24) :: compnames(maxc)`
    *   **Purpose:** Facilitates returning multiple values (component count and an array of names) from `pytqgcom` to Python in a structured way.

## Wrapped Subroutines and Functions

The subroutines generally mirror those in `liboctq`, with a `py` prefix. `!f2py` directives are not explicitly visible in this file but are implied by its structure and intent for `f2py` processing.

*   **`pygeterr() result(errorcode)` (Function)**
    *   **Description:** Retrieves the current error code from the underlying Fortran library (`gx%bmperr`).
    *   **Returns:** `errorcode` (Integer).

*   **`pyseterr(errorcode)` (Subroutine)**
    *   **Description:** Sets the error code in the underlying Fortran library.
    *   **Parameters/Arguments:** `errorcode` (Integer, IN).

*   **`pytqini(n, eq)`**
    *   **Description:** Wraps `tqini`. Initializes the TQ interface.
    *   **Parameters/Arguments:** `n` (Integer, IN), `eq` (Type `eq_wrapper`, OUT).

*   **`pytqrfil(filename, eq)`**
    *   **Description:** Wraps `tqrfil`. Reads a thermodynamic database.
    *   **Parameters/Arguments:** `filename` (Character, IN), `eq` (Type `eq_wrapper`, INOUT). The `eq%ceq` pointer is passed to `tqrfil`.

*   **`pytqrpfil(filename, nsel, selel, eq)`**
    *   **Description:** Wraps `tqrpfil`. Reads a database with selected elements.
    *   **Parameters/Arguments:** `filename` (Character, IN), `nsel` (Integer, IN), `selel` (Character array, IN), `eq` (Type `eq_wrapper`, INOUT).

*   **`pytqgcom(comp, eq)`**
    *   **Description:** Wraps `tqgcom`. Gets system component names and count.
    *   **Parameters/Arguments:** `comp` (Type `comp_wrapper`, OUT), `eq` (Type `eq_wrapper`, INOUT).

*   **`pytqce(target, nn1, nn2, value, eq)`**
    *   **Description:** Wraps `tqce`. Calculates equilibrium.
    *   **Parameters/Arguments:** `target` (Character, IN), `nn1`, `nn2` (Integer, IN), `value` (Double precision, INOUT), `eq` (Type `eq_wrapper`, INOUT).

*   **`pytqgetv(stavar, nn1, nn2, nn3in, nn3out, values, eq)`**
    *   **Description:** Wraps `tqgetv`. Retrieves calculation results. `nn3in` is the input dimension of `values`, `nn3out` is the actual number of values returned by `tqgetv`.
    *   **Parameters/Arguments:** `stavar` (Character, IN), `nn1`, `nn2`, `nn3in` (Integer, IN), `nn3out` (Integer, OUT), `values` (Double precision array, INOUT), `eq` (Type `eq_wrapper`, INOUT).

*   **Other `py*` routines:**
    *   `pytqgnp`, `pytqgpn`, `pytqgpi`, `pytqgpi2`, `pytqgpcn2`, `pytqgpcs`, `pytqphsts`, `pytqphsts2`, `pytqsetc`, `pytqtgsw`, `pytqgphc1`, `pytqsphc1`, `pytqcph1`, `pytqcph2`, `pytqcph3`, `pytqdceq`, `pytqcceq`, `pytqselceq`, `pytqcref`, `pytqlr`, `pytqlc`, `pytqquiet`.
    *   **Description:** These subroutines wrap their corresponding counterparts in `liboctq` (e.g., `pytqgnp` calls `tqgnp`). They use the `eq_wrapper` to pass the equilibrium data structure pointer.

# Important Variables/Constants

*   **Module `RawOpenCalphad`:** The Fortran module itself, which will become the name of the Python module (or part of it) after `f2py` processing.
*   **`eq_wrapper` (Derived Type):** As described above, this is crucial for `f2py` to correctly handle the `gtp_equilibrium_data` pointer, making it usable from Python.
*   **`comp_wrapper` (Derived Type):** As described above, bundles component data for f2py.
*   The parameters like `maxc` are implicitly used from `liboctq` when defining `comp_wrapper`.

# Usage Examples

This Fortran file (`pyOC.f90`) is not intended to be compiled and used directly from other Fortran code. Instead, it is a source file for `f2py`, a tool that generates Python extension modules from Fortran (or C) code.

**Typical `f2py` compilation process:**

```bash
# Example f2py command (may need to include other source files or libraries from OpenCalphad)
f2py -c -m pyOC RawOpenCalphad.f90 liboctq.F90 [other_dependencies.F90 ...] -L/path/to/libs -lsomelib
```

Once compiled, the `RawOpenCalphad` module (or whatever name is specified with `-m` in f2py, e.g. `pyOC`) can be imported and used in Python:

```python
# Conceptual Python usage example
# import pyOC # Or the name given to f2py's -m flag

# try:
#     eq_data = pyOC.pytqini(0) # eq_data is an f2py object wrapping eq_wrapper
#     pyOC.pytqrfil("mydatabase.tdb", eq_data)
#     err_code = pyOC.pygeterr()
#     if err_code != 0:
#         print(f"Error after tqrfil: {err_code}")
#         # Potentially call a function to get error message string
#
#     # Example of setting a condition: Temperature to 1000 K
#     # Assuming cnum_out is an integer to hold the condition number
#     # Need to handle how f2py exposes cnum (might be part of a tuple return)
#     # pyOC.pytqsetc("T", 0, 0, 1000.0, eq_data) # Actual f2py signature might vary for cnum
#
#     # ... further calls ...
#
# except Exception as e:
#     print(f"An error occurred: {e}")
#     err_code = pyOC.pygeterr()
#     print(f"OC Error Code: {err_code}")

```

# Dependencies and Interactions

*   **`liboctq` (Fortran Module):** This is the primary functional dependency. The `pyOC.f90` module contains wrappers that call the actual thermodynamic routines in `liboctq`. The `use liboctq` statement makes these routines accessible.
*   **`f2py` (Tool):** This file is specifically structured for processing by `f2py`. `f2py` reads the Fortran code, including the `py*` wrapper routines and the derived types, and generates C code that acts as the glue between Python and Fortran. This C code is then compiled into a Python extension module (`.so` or `.pyd` file).
*   **Python Runtime Environment:** The module generated by `f2py` is loaded and used within a Python interpreter. The `eq_wrapper` and `comp_wrapper` types are exposed to Python as custom f2py-generated types.
*   **Compilation Environment:** A Fortran compiler (compatible with `f2py`) and a C compiler (for the f2py-generated C code) are needed, along with the Python development headers.
*   The comment at the top mentions "OCASI interface (liboctq.F90)". It also mentions `f90wrapp-compatible`, which might refer to a specific set of conventions or an older tool/style for creating f2py wrappers.
