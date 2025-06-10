# Overview

The Fortran file `src/models/gtp3B.F90` is a vital part of the General Thermodynamic Package (GTP) in OpenCalphad, included within the main `GENERAL_THERMODYNAMIC_PACKAGE` module (defined in `gtp3.F90`). As indicated by comments in `gtp3.F90` ("12: enter data"), this file's primary role is to provide the subroutines necessary for entering and defining thermodynamic data within the GTP system. This includes creating records for elements, species, phases (along with their crystallographic and model details), constituents within phases, and the various model parameters that describe their thermodynamic behavior.

These routines are fundamental for populating the internal GTP data structures, typically when parsing thermodynamic database files (e.g., TDB format) or when a user programmatically defines a thermodynamic system.

# Key Components

This file contains a suite of subroutines designed to create and populate different aspects of the thermodynamic database.

## Entity Definition Subroutines

*   **`store_element(symb, name, refstate, mass, h298, s298)`**
    *   **Description:** Creates a new element record in the GTP system. It checks for valid input and ensures the element symbol is unique. A corresponding species record for the element is also automatically created.
    *   **Parameters/Arguments:** `symb` (Symbol, e.g., "FE"), `name` (Full name, e.g., "IRON"), `refstate` (Reference state string), `mass` (Atomic mass), `h298` (Enthalpy at 298.15K relative to 0K), `s298` (Entropy at 298.15K).

*   **`enter_species(symb, noelx, ellist, stoik)`**
    *   **Description:** Creates a new species record. A species can be an element, an ion, or a molecule.
    *   **Parameters/Arguments:** `symb` (Species name/symbol), `noelx` (Number of elements in the species), `ellist` (Array of element symbols forming the species, "/-" for electron), `stoik` (Array of stoichiometric factors for each element).

*   **`enter_phase(name, nsl, knr, const, sites, model, phtype, warning, emodel)` / `enter_phase_old(...)`**
    *   **Description:** Defines a new phase in the system. This is a complex routine that sets up the phase name, number of sublattices (`nsl`), number of constituents in each sublattice (`knr`), names of constituents (`const`), site ratios for each sublattice (`sites`), the thermodynamic model name (`model`, e.g., "CEF", "I2SL", "MQMQA"), and the physical phase type (`phtype`: G, L, S). It also handles entropy model specifications (`emodel`). It initializes the phase's data structures, including its first composition set. `enter_phase_old` is likely a variant or older version.
    *   **Parameters/Arguments:** As listed above. `warning` is a logical flag.

*   **`enter_cvmtfs_phase(name, nsl, knr, const)`**
    *   **Description:** A specialized routine for defining phases that use the Cluster Variation Method (CVM) for a tetrahedron FCC structure with Short-Range Order (SRO). It automatically generates the necessary SRO cluster constituents based on the input elements.
    *   **Parameters/Arguments:** `name` (Phase name), `nsl` (Number of sublattices, must be 1), `knr` (Number of elements), `const` (Array of element names).

*   **`enter_composition_set(iph, prefix, suffix, icsno)`**
    *   **Description:** Adds a new composition set to an existing phase `iph`. This allows a single phase to have multiple configurations (e.g., ordered and disordered states, or different magnetic states) treated as distinct composition sets.
    *   **Parameters/Arguments:** `iph` (Phase index), `prefix` (Optional name prefix), `suffix` (Optional name suffix), `icsno` (Returned composition set index).

*   **`enter_parameter(lokph, typty, fractyp, nsl, endm, nint, lint, ideg, lfun, refx)`**
    *   **Description:** This is a central routine for entering thermodynamic model parameters for a given phase (`lokph`). It handles parameters for endmembers and interactions of various orders (`nint`, `lint`, `ideg`).
    *   **Parameters/Arguments:** `lokph` (Phase location), `typty` (Property type, e.g., G, TC, MQ), `fractyp` (Fraction type, site or disordered - though disordered seems less used here directly), `nsl` (Number of sublattices involved in parameter), `endm` (Array of constituent indices defining the endmember/interaction vertex), `nint` (Number of interacting constituents), `lint` (Array defining interacting constituents), `ideg` (Degree of the parameter, e.g., 0 for L0, 1 for L1), `lfun` (Link to TP function for the parameter value), `refx` (Bibliographic reference key).

*   **`enter_equilibrium(name, number)` / `copy_equilibrium(neweq, name, ceq)` / `copy_equilibrium2(...)`**
    *   **Description:** Manage equilibrium records. `enter_equilibrium` creates a new, named equilibrium record. `copy_equilibrium` and `copy_equilibrium2` create a new equilibrium record by duplicating an existing one (`ceq`), including its conditions and results.

*   **`enter_many_equil(cline, last, pun)`**
    *   **Description:** A powerful utility to define and create multiple equilibrium records based on a common template of commands (like fixed phases, conditions, experiments) and a table of varying values. This is useful for batch calculations or setting up series of related equilibria.

## Data Handling and Utility Subroutines

*   **`sort_ionliqconst(lokph, mode, knr, kconlok, klok)`**: Sorts the constituents of an ionic liquid phase according to specific rules (cations first, then anions, vacancies, and neutrals). This is important for the correct application of the ionic liquid model.
*   **`remove_composition_set(iph, force)` / `suspend_composition_set(iph, parallel, ceq)` / `suspend_unstable_sets(mode, ceq)`**: Routines for managing composition sets by deleting them or changing their status to suspended.
*   **`fccpermuts(...)` / `bccpermuts(...)` (and associated helpers like `fccint31`, `fccint22`, `fccpe211`, `bccendmem`, `bccint1`, `bccint2`)**: These subroutines are responsible for generating the symmetrically equivalent parameters for phases with FCC or BCC crystal structures that exhibit ordering (specified by TYPE_DEFINITION options F or B in TDB files). Given one parameter, these routines determine all other parameters that must be equal due to crystal symmetry.
*   **`tdbrefs(refid, line, mode, iref)`**: Stores or amends bibliographic reference entries, typically parsed from TDB files.
*   **`mqmqa_constituents(...)` / `mqmqa_species(...)` / `mqmqa_rearrange(...)`**: Specialized routines for handling the complex setup of constituents (quadruplets, pairs) and their stoichiometric relations for phases modeled with the Modified Quasichemical Model in the Quadruplet Approximation (MQMQA).
*   **`amend_components(...)`**: Allows changing the set of system components, which involves updating stoichiometry matrices.

# Important Variables/Constants

This file primarily acts on the global data structures of GTP, which are defined in included files like `gtp3_dd1.F90` and `gtp3_dd2.F90`. These include:
*   Arrays for storing element (`ellista`), species (`splista`), phase (`phlista`), and phase tuple (`phasetuple`) data.
*   Counters for these entities (`noofel`, `noofsp`, `noofph`, `nooftuples`).
*   Equilibrium records (`eqlista`).
*   Pointers to manage lists of parameters within phases (e.g., `phlista(lokph)%ordered`, `phlista(lokph)%disordered`).
*   Parameters from `ocparam.F90` defining maximum sizes of arrays (e.g., `maxel`, `maxsp`, `maxph`, `maxconst`).

# Usage Examples

The subroutines in `gtp3B.F90` are predominantly used by the parts of OpenCalphad that parse thermodynamic databases. For example:
*   When a TDB file is read (by routines in `gtp3E.F90`), commands like `ELEMENT`, `SPECIES`, `PHASE`, `CONSTITUENT`, and `PARAMETER` are processed. This processing involves calls to `store_element`, `enter_species`, `enter_phase`, and `enter_parameter` respectively, to populate GTP's internal data structures.
*   The `enter_many_equil` command in the OpenCalphad command-line interface directly uses the `enter_many_equil` subroutine from this file.
*   Programmatic interfaces or scripts that define thermodynamic systems from scratch would also use these "enter" routines.

# Dependencies and Interactions

*   **Core GTP Data Structures:** This file directly populates and modifies the primary data arrays and derived types that constitute the thermodynamic database within GTP (defined in `gtp3_dd1.F90`, `gtp3_dd2.F90`).
*   **Utility Functions:** Uses helper functions from other GTP components or `metlib` for tasks like string manipulation (`capson`), name checking (`proper_symbol_name`), and error handling.
*   **Database Parsing Modules (e.g., `gtp3E.F90`, `gtp3EX.F90`):** These modules are the primary callers of the data entry routines in `gtp3B.F90`.
*   **Calculation Modules (e.g., `gtp3X.F90`, `matsmin.F90`):** The data entered via `gtp3B.F90` routines forms the basis upon which all subsequent thermodynamic calculations and Gibbs energy minimizations are performed.

This file is critical for constructing the in-memory representation of the thermodynamic system that OpenCalphad works with.
