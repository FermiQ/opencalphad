# Overview

This file, `liboctq.F90`, provides a minimal TQ (Thermo-Calc Query) interface for the OpenCalphad software. It acts as a bridge, allowing other applications to interact with the core OpenCalphad functionalities for thermodynamic and phase equilibrium calculations. To be used, it typically needs to be compiled and linked against the main OpenCalphad library (e.g., `oclib.a`).

# Key Components

This module defines several subroutines and functions to interact with the OpenCalphad system. The primary components are:

## Initialization and File Reading

*   **`tqini(n, ceq)`**
    *   **Description:** Initiates the OpenCalphad workspace. `ceq` is a pointer to the current equilibrium data.
    *   **Parameters/Arguments:** `n` (Integer, not actively used), `ceq` (Type `gtp_equilibrium_data`, pointer, EXIT: current equilibrium).
    *   **Returns:** None directly (modifies `ceq`).

*   **`tqrfil(filename, ceq)`**
    *   **Description:** Reads all elements and other data from a Thermo-Calc Database (TDB) file.
    *   **Parameters/Arguments:** `filename` (Character, IN: database filename), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqrpfil(filename, nsel, selel, ceq)`**
    *   **Description:** Reads data from a TDB file, but only for a selected list of elements.
    *   **Parameters/Arguments:** `filename` (Character, IN: database filename), `nsel` (Integer, IN: number of selected elements), `selel` (Character array, IN: selected elements), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

## System Information Retrieval

*   **`tqgcom(n, compnames, ceq)`**
    *   **Description:** Gets the names and number of system components (currently, these are the elements).
    *   **Parameters/Arguments:** `n` (Integer, EXIT: number of components), `compnames` (Character array, EXIT: names of components), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqgnp(n, ceq)`**
    *   **Description:** Gets the total number of phase tuples. A phase tuple represents a phase and its specific composition set.
    *   **Parameters/Arguments:** `n` (Integer, EXIT: number of phase tuples), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqgpn(phtupx, phasename, ceq)`**
    *   **Description:** Gets the name of a phase tuple given its index.
    *   **Parameters/Arguments:** `phtupx` (Integer, IN: phase tuple index), `phasename` (Character, EXIT: phase name), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqgpi(phtupx, phasename, ceq)`**
    *   **Description:** Gets the phase tuple index for a given phase name (including its composition set).
    *   **Parameters/Arguments:** `phtupx` (Integer, EXIT: phase tuple index), `phasename` (Character, IN: phase name), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqgpi2(iph, ics, phasename, ceq)`**
    *   **Description:** Gets the phase index (`iph`) and composition set index (`ics`) for a given phase name.
    *   **Parameters/Arguments:** `iph` (Integer, EXIT: phase index), `ics` (Integer, EXIT: composition set index), `phasename` (Character, IN: phase name), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: current equilibrium).
    *   **Returns:** None.

*   **`tqgpcn2(n, c, csname)`**
    *   **Description:** Gets the name of a constituent `c` in phase `n`.
    *   **Parameters/Arguments:** `n` (Integer, IN: phase number), `c` (Integer, IN: constituent index), `csname` (Character, EXIT: constituent name).
    *   **Returns:** None.

*   **`tqgpci(n, c, constituentname, ceq)`**
    *   **Description:** Gets the index of a constituent by its name within a phase. (Marked as "not implemented yet").
    *   **Parameters/Arguments:** `n` (Integer, IN: phase index), `c` (Integer, IN: sequential constituent index), `constituentname` (Character, IN), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqgpcs(c, nspel, ielno, stoi, smass, qsp)`**
    *   **Description:** Gets details of a constituent `c`, such as its stoichiometry, mass, and charge.
    *   **Parameters/Arguments:** `c` (Integer, IN: sequential constituent index), `nspel` (Integer, EXIT: number of elements in species), `ielno` (Integer array, EXIT: element indices), `stoi` (Double precision array, EXIT: stoichiometry), `smass` (Double precision, EXIT: mass), `qsp` (Double precision, EXIT: charge).
    *   **Returns:** None.

*   **`tqgccf(n1, n2, elnames, stoi, mass, ceq)`**
    *   **Description:** Gets the stoichiometry of a system component `n1`. (Marked as "not implemented yet").
    *   **Parameters/Arguments:** `n1` (Integer, IN: component number), `n2` (Integer, EXIT: number of elements in component), `elnames` (Character array, EXIT: element symbols), `stoi` (Double precision array, EXIT: stoichiometry), `mass` (Double precision, EXIT: component mass), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqgnpc(n, c, ceq)`**
    *   **Description:** Gets the number of constituents in phase `n`. (Marked as "not implemented yet").
    *   **Parameters/Arguments:** `n` (Integer, IN: phase number), `c` (Integer, EXIT: number of constituents), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

## Setting Conditions and Status

*   **`tqcref(ciel, phase, tpref, ceq)`**
    *   **Description:** Sets the reference state for a component.
    *   **Parameters/Arguments:** `ciel` (Integer, IN: component index), `phase` (Character, IN: phase name for reference), `tpref` (Double precision array, IN: T, P for reference state), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqphsts(phtupx, newstat, val, ceq)`**
    *   **Description:** Sets the status of a phase tuple (e.g., SUSPEND, DORMANT, ENTERED, FIX).
    *   **Parameters/Arguments:** `phtupx` (Integer, IN: phase tuple index, or <=0 for all), `newstat` (Integer, IN: new status), `val` (Double precision, IN: value associated with status change), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqphsts2(phnames, newstat, val, ceq)`**
    *   **Description:** Sets the status for multiple phases at once using a string of phase names.
    *   **Parameters/Arguments:** `phnames` (Character, IN: comma-separated phase names), `newstat` (Integer, IN: new status), `val` (Double precision, IN: value), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqsetc(stavar, n1, n2, value, cnum, ceq)`**
    *   **Description:** Sets a condition for equilibrium calculation (e.g., temperature, pressure, composition).
    *   **Parameters/Arguments:** `stavar` (Character, IN: state variable symbol), `n1` (Integer, IN: phase tuple or component index), `n2` (Integer, IN: component index or aux), `value` (Double precision, IN: condition value), `cnum` (Integer, EXIT: condition index), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`reset_conditions(cline, ceq)`**
    *   **Description:** Resets conditions, typically for temperature.
    *   **Parameters/Arguments:** `cline` (Character, IN: condition string e.g., 'T=NONE'), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`Change_Status_Phase(myname, nystat, myval, ceq)`**
    *   **Description:** Changes the status of a named phase.
    *   **Parameters/Arguments:** `myname` (Character, IN: phase name), `nystat` (Integer, IN: new status), `myval` (Double precision, IN: value), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

## Equilibrium Calculation and Results

*   **`tqce(target, n1, n2, value, ceq)`**
    *   **Description:** Calculates equilibrium. Can have a target for specific calculations. `n1 < 0` implies calculation without grid minimizer.
    *   **Parameters/Arguments:** `target` (Character, IN: target variable for calculation), `n1` (Integer, IN: mode control/index), `n2` (Integer, IN: index), `value` (Double precision, EXIT: calculated value of target), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None (updates `ceq` and potentially `value`).

*   **`tqgetv(stavar, n1, n2, n3, values, ceq)`**
    *   **Description:** Retrieves results from an equilibrium calculation using state variable symbols.
    *   **Parameters/Arguments:** `stavar` (Character, IN: state variable symbol), `n1`, `n2` (Integer, IN: indices for variable), `n3` (Integer, IN/OUT: dimension of `values`/number of values returned), `values` (Double precision array, EXIT: result(s)), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

## Phase Constitution and Properties

*   **`tqgphc1(n1, nsub, cinsub, spix, yfrac, sites, extra, ceq)`**
    *   **Description:** Gets the phase constitution (sublattices, constituents, site fractions, etc.) for a phase tuple `n1`.
    *   **Parameters/Arguments:** `n1` (Integer, IN: phase tuple index), `nsub` (Integer, EXIT: number of sublattices), `cinsub` (Integer array, EXIT: constituents per sublattice), `spix` (Integer array, EXIT: species indices), `yfrac` (Double precision array, EXIT: constituent fractions), `sites` (Double precision array, EXIT: site ratios), `extra` (Double precision array, EXIT: extra values like moles/formula unit), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqsphc1(n1, yfra, extra, ceq)`**
    *   **Description:** Sets the phase constitution for a phase tuple `n1`.
    *   **Parameters/Arguments:** `n1` (Integer, IN: phase tuple index), `yfra` (Double precision array, IN: constituent fractions), `extra` (Double precision array, EXIT: calculated extra values), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqcph1(n1, n2, n3, gtp, dgdy, d2gdydt, d2gdydp, d2gdy2, ceq)`**
    *   **Description:** Calculates phase properties (G, derivatives w.r.t T, P, composition) for a phase tuple `n1` at its current constitution. Returns results in separate arrays.
    *   **Parameters/Arguments:** `n1` (Integer, IN: phase tuple index), `n2` (Integer, IN: calculation type 0,1,2), `n3` (Integer, EXIT: number of constituents), `gtp` (Double precision array(6), EXIT: G and T/P derivatives), `dgdy`, `d2gdydt`, `d2gdydp`, `d2gdy2` (Double precision arrays, EXIT: composition derivatives), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqcph2(n1, n2, n3, n4, ceq)`**
    *   **Description:** Similar to `tqcph1`, but returns an index `n4` to an internal data structure (`ceq%phase_varres`) where results are stored.
    *   **Parameters/Arguments:** `n1` (Integer, IN: phase tuple index), `n2` (Integer, IN: calculation type), `n3` (Integer, EXIT: number of constituents), `n4` (Integer, EXIT: index to results), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqcph3(n1, n2, g, ceq)`**
    *   **Description:** Similar to `tqcph1`, but returns all G derivatives in a single array `g`.
    *   **Parameters/Arguments:** `n1` (Integer, IN: phase tuple index), `n2` (Integer, IN: calculation type), `g` (Double precision array, EXIT: G and its derivatives), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqgdmat(phtupx, tpval, xknown, cpot, tyst, nend, mugrad, mobval, consnames, n1, ceq)`**
    *   **Description:** Equilibrates constituent fractions of a phase for given mole fractions `xknown` and calculates diffusion-related quantities.
    *   **Parameters/Arguments:** `phtupx` (Integer, IN: phase tuple index), `tpval` (Double array, IN: T and P), `xknown` (Double array, IN: mole fractions), `cpot` (Double array, EXIT: chemical potentials), `tyst` (Logical, IN: suppress output), `nend` (Integer, EXIT: number of values in `mugrad`), `mugrad` (Double array, EXIT: mu gradients), `mobval` (Double array, EXIT: mobilities), `consnames` (Character array, EXIT: constituent names), `n1` (Integer, EXIT: num constituents), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

## Equilibrium Record Management

*   **`tqdceq(name)`**
    *   **Description:** Deletes a stored equilibrium record by its name.
    *   **Parameters/Arguments:** `name` (Character, IN: name of equilibrium to delete).
    *   **Returns:** None.

*   **`tqcceq(name, n1, newceq, ceq)`**
    *   **Description:** Copies the current equilibrium data (`ceq`) to a new equilibrium record (`newceq`) with a given name.
    *   **Parameters/Arguments:** `name` (Character, IN: name for the new equilibrium), `n1` (Integer, EXIT: index of new equilibrium), `newceq` (Type `gtp_equilibrium_data`, pointer, EXIT: new equilibrium record), `ceq` (Type `gtp_equilibrium_data`, pointer, IN: source equilibrium).
    *   **Returns:** None.

*   **`tqselceq(name, ceq)`**
    *   **Description:** Selects an existing equilibrium record by its name to be the current one.
    *   **Parameters/Arguments:** `name` (Character, IN: name of equilibrium to select), `ceq` (Type `gtp_equilibrium_data`, pointer, EXIT: selected equilibrium).
    *   **Returns:** None.

## Utility and Debugging

*   **`tqlr(lut, ceq)`**
    *   **Description:** Lists equilibrium results to a specified output unit `lut` (typically for debugging).
    *   **Parameters/Arguments:** `lut` (Integer, IN: output unit number), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqlc(lut, ceq)`**
    *   **Description:** Lists current conditions to a specified output unit `lut` (for debugging).
    *   **Parameters/Arguments:** `lut` (Integer, IN: output unit number), `ceq` (Type `gtp_equilibrium_data`, pointer, IN).
    *   **Returns:** None.

*   **`tqtgsw(i)`**
    *   **Description:** Toggles the status of a global status word bit `i`.
    *   **Parameters/Arguments:** `i` (Integer, IN: bit index to toggle).
    *   **Returns:** None.

*   **`tqquiet(yes)`**
    *   **Description:** Suppresses or enables verbose output from the library.
    *   **Parameters/Arguments:** `yes` (Logical, IN: .TRUE. to suppress output, .FALSE. to enable).
    *   **Returns:** None.

# Important Variables/Constants

The `liboctq` module defines several key variables and constants:

*   **`maxc` (Parameter):** An integer parameter, initialized from `maxel` (maximum elements) obtained from the `liboceqplus` module. Defines the maximum number of components the system can handle.
*   **`maxp` (Parameter):** An integer parameter, initialized from `maxph` (maximum phases) obtained from the `liboceqplus` module. Defines the maximum number of phases.
*   **`nel` (Integer):** Stores the actual number of elements (components) in the current system, determined after reading a database.
*   **`cnam(maxc)` (Character array):** An array of characters (strings of length 24) to store the names of the system components (elements).
*   **`cmass(maxc)` (Double precision array):** An array to store the masses of the components. (Its direct usage within `liboctq` module scope isn't prominent, may be used by routines from `liboceqplus`).
*   **`ntup` (Integer):** Stores the current number of phase tuples in the system. A phase tuple uniquely identifies a phase and its composition set. This number can change as new composition sets are formed or (potentially) deleted.
*   **`ysave(:,:)` (Allocatable Double precision array):** Used to save phase constitutions, potentially to speed up subsequent calculations by interpolation if constitutions change minimally.

# Usage Examples

The Fortran source file contains comments like `!\begin{verbatim}` which show subroutine signatures and argument lists. However, these are primarily for defining the interface. Specific, self-contained runnable usage examples are not readily available within the file's comments. To use these routines, one would typically:
1. Initialize the system with `tqini`.
2. Read a thermodynamic database with `tqrfil` or `tqrpfil`.
3. Set conditions (T, P, composition) using `tqsetc`.
4. Perform an equilibrium calculation with `tqce`.
5. Retrieve desired results using `tqgetv` or phase-specific functions.

```fortran
! Conceptual example (not directly from file comments)
! PROGRAM MY_CALCULATION
!   USE liboctq
!   IMPLICIT NONE
!   TYPE(gtp_equilibrium_data), POINTER :: eq
!   INTEGER :: ncomp, nphasetuples, err
!   CHARACTER(LEN=24), DIMENSION(:), ALLOCATABLE :: compnames
!
!   CALL tqini(0, eq)
!   CALL tqrfil('my_database.tdb', eq)
!   IF (gx%bmperr /= 0) STOP 'Error reading TDB'
!
!   CALL tqgcom(ncomp, compnames, eq) ! Error: compnames needs to be allocated first
!   ! ... further calls to set conditions, calculate equilibrium, get results ...
! END PROGRAM MY_CALCULATION
```
*(Note: The Fortran example above is a conceptual illustration and may require adjustments to be fully functional, such as memory allocation for `compnames` before calling `tqgcom`.)*

# Dependencies and Interactions

*   **`liboceqplus`:** This module heavily relies on `liboceqplus`, as indicated by the `use liboceqplus` statement. `liboceqplus` provides access to the main OpenCalphad library routines for equilibrium calculations, thermodynamic models, and data management (like `gtp_equilibrium_data`, `phasetuple`, etc.).
*   **Compilation:** The initial comments in `liboctq.F90` state: "To compile and link this with an application one must first compile and form a library with of the most OC subroutines (oclib.a) and to copy this and the corresponding "mov" files from this compilation to the folder with this library". This highlights a build dependency on the compiled OpenCalphad core library.
*   **Phase Tuples:** The interface uses a Fortran `TYPE` called `gtp_phasetuple` (defined in `liboceqplus`) for identifying phases and their composition sets. The number and indexing of these tuples can change during calculations, which is an important interaction detail.
