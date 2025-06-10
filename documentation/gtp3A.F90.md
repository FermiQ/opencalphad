# Overview

The file `src/models/gtp3A.F90` is a core component of the General Thermodynamic Package (GTP) within OpenCalphad. It is included in the main `gtp3.F90` file, which defines the `GENERAL_THERMODYNAMIC_PACKAGE` module. This specific part of GTP, `gtp3A.F90`, focuses on the fundamental operations of initializing the thermodynamic system, managing core data structures, and providing a suite of functions to query (count, find, get) and set various thermodynamic entities and their properties.

It lays the groundwork for defining the system (elements, species, phases) and provides essential accessor and manipulator routines used by other parts of OpenCalphad, such as the Gibbs energy minimizer and database interface modules.

# Key Components

This file contains numerous subroutines and functions. The comments in `gtp3.F90` categorize them into:
1.  Initialization and re-initiation
2.  Number of things (querying counts)
3.  Find things (locating specific entities)
4.  Get things (retrieving data)
5.  Set things (modifying data or status)

Key subroutines and functions include (but are not limited to):

## Initialization and Data Management

*   **`init_gtp(intvar, dblvar)`**
    *   **Description:** Initializes the core GTP data structures. This includes allocating arrays for elements, species, phases, phase tuples, bibliographic references, property identifiers, and equilibrium records. It sets up predefined entities like the electron and vacancy elements/species and default TP/state variable functions (R, RT, T_C). It also initializes global data such as the gas constant.
    *   **Parameters/Arguments:** `intvar(*)` (Integer array, potentially for future allocation control or default integer settings), `dblvar(*)` (Double precision array, potentially for default real settings).

*   **`initialize_default_global_parameters(firsteq)`**
    *   **Description:** Sets default values for various global parameters associated with an equilibrium record (e.g., `gmindif` for grid minimizer, `xconv` for convergence criteria, `maxiter` for maximum iterations).
    *   **Parameters/Arguments:** `firsteq` (Pointer to `gtp_equilibrium_data`).

*   **`new_gtp()`**
    *   **Description:** Resets the entire GTP system by deallocating and deleting all existing thermodynamic data including elements, species, phases, equilibria, TP functions, and state variable functions. This is essential for reading a new database or starting a completely new calculation. It internally calls `delphase` and other cleanup routines.

*   **`deallocate_gtp(intvar, dblvar)`**
    *   **Description:** Deallocates several of the main arrays used by GTP, effectively freeing memory associated with the core data structures.

*   **`delphase(lokph)`**
    *   **Description:** Deallocates all data specifically associated with a given phase identified by its location `lokph`. This includes endmember parameters, interaction parameters, and other phase-specific properties.

## Querying Functions (Number of Entities)

*   **`noel() RESULT(integer)`**: Returns the current number of elements defined in the system.
*   **`nosp() RESULT(integer)`**: Returns the current number of species.
*   **`noph() RESULT(integer)`**: Returns the current number of phases.
*   **`noofcs(iph) RESULT(integer)`**: Returns the number of composition sets for a given phase `iph`.
*   **`noconst(iph, ics, ceq) RESULT(integer)`**: Returns the number of constituents in phase `iph`, composition set `ics`, considering the current equilibrium `ceq`.
*   **`nooftup() RESULT(integer)`**: Returns the total number of active phase tuples.
*   **`nosvf() RESULT(integer)`**: Returns the number of defined state variable functions.
*   **`noeq() RESULT(integer)`**: Returns the number of entered equilibrium records.
*   **`nonsusphcs(ceq) RESULT(integer)`**: Returns the total number of unhidden and unsuspended phases + composition sets.

## Querying Functions (Finding Entities)

*   **`find_element_by_name(name, iel)`**: Finds the alphabetical index `iel` of an element given its exact `name`.
*   **`find_component_by_name(name, icomp, ceq)`**: Finds the index `icomp` of a system component (which is a species) by its `name` within the context of equilibrium `ceq`.
*   **`find_species_by_name(name, isp)` / `find_species_by_name_exact(name, isp)`**: Find the alphabetical index `isp` of a species by its `name`, allowing for unique abbreviations or requiring an exact match.
*   **`find_species_record(name, loksp)` / `find_species_record_noabbr(name, loksp)` / `find_species_record_exact(name, loksp)`**: Find the storage location `loksp` of a species record by its `name`.
*   **`find_phasetuple_by_name(name, phcsx)`**: Finds the index `phcsx` of a phase tuple given its full `name` (which can include prefixes/suffixes or #digit for composition sets). Calls `find_phasex_by_name`.
*   **`find_phase_by_name(name, iph, ics)`**: Finds phase index `iph` and composition set index `ics` by name. Calls `find_phasex_by_name`.
*   **`find_phasetuple_by_indices(iph, ics) RESULT(integer)`**: Returns phase tuple index corresponding to a given phase index `iph` and composition set index `ics`.
*   **`find_phasex_by_name(name, phcsx, iph, zcs)`**: A core routine for finding a phase. It returns the phase tuple index `phcsx`, alphabetical phase index `iph`, and composition set index `zcs` based on the input `name`, correctly parsing complex names with prefixes, suffixes, and #digit for composition sets.
*   **`find_constituent(iph, spname, mass, icon)`**: Finds the sequential index `icon` and `mass` of a constituent within phase `iph` based on its `spname` (which can include a #sublattice specifier).
*   **`findeq(name, ieq)`**: Finds the index `ieq` of an equilibrium record by its `name`.
*   **`selecteq(ieq, ceq)`**: Sets the current equilibrium pointer `ceq` to the equilibrium record `ieq`.

## Querying Functions (Getting Data)

*   **`get_element_data(iel, elsym, elname, refstat, mass, h298, s298)`**: Retrieves various data for an element `iel`, such as its symbol `elsym`, full name `elname`, reference state `refstat`, atomic `mass`, and standard enthalpy/entropy.
*   **`get_component_name(icomp, name, ceq)`**: Retrieves the `name` of component `icomp` in the context of equilibrium `ceq`.
*   **`get_species_name(isp, spsym)` / `get_species_location(isp, loksp, spsym)`**: Retrieves the symbol `spsym` (and optionally location `loksp`) of a species `isp`.
*   **`get_species_data(...)` / `get_species_component_data(...)`**: Retrieve detailed stoichiometry, mass, charge for a species (element or component basis).
*   **`get_phase_name(iph, ics, name)`**: Constructs the full, potentially decorated (with pre/suffix, #cs) `name` of phase `iph`, composition set `ics`.
*   **`get_phasetup_name(phtupx, name)` / `get_phasetuple_name(phtuple, name)`**: Constructs the full phase name from a phase tuple index `phtupx` or a `gtp_phasetuple` record.
*   **`get_phase_data(iph, ics, nsl, nkl, knr, yarr, sites, qq, ceq)`**: Retrieves comprehensive data for phase `iph`, composition set `ics`, including number of sublattices `nsl`, constituents per sublattice `nkl`, species locations `knr`, constituent fractions `yarr`, site ratios `sites`, and derived quantities `qq` (like atoms per formula unit, net charge).

## Setting Functions

*   **`set_constitution(iph, ics, yfra, qq, ceq)`**: Sets the constituent fractions `yfra` for phase `iph`, composition set `ics`. It also recalculates and updates related properties like the number of atoms per formula unit and net charge (returned in `qq`).
*   **`set_reference_state(icomp, iph, tpval, ceq)`**: Sets the reference state for component `icomp` to be phase `iph` at the specified temperature and pressure `tpval(2)`. This involves finding a suitable endmember of the reference phase that matches the component's stoichiometry.

*(Note: Functions like `set_condition`, `change_phtup_status`, `test_phase_status`, and `get_state_var_value`, while crucial for GTP operations, are primarily defined in other `gtp3*.F90` files as per the include structure in `gtp3.F90` but are used by or interact with routines in `gtp3A.F90`.)*

# Important Variables/Constants

While most data is stored within derived types managed by these routines, the core of GTP revolves around module-level arrays and counters for elements, species, phases, etc. These are typically defined in the included `gtp3_dd1.F90` and `gtp3_dd2.F90` files. Examples include:
*   `ellista`, `elements`: For storing element data.
*   `splista`, `species`: For storing species data.
*   `phlista`, `phases`: For storing phase data.
*   `phasetuple`: For storing phase tuple data.
*   `noofel`, `noofsp`, `noofph`, `nooftuples`: Counters for these entities.
*   `firsteq`: Pointer to the first (default) equilibrium record.
*   `globaldata`: A derived type instance holding global settings like the gas constant.

# Usage Examples

The subroutines and functions in `gtp3A.F90` are fundamental internal components of the OpenCalphad system. They are not typically called directly by an end-user but are invoked by other higher-level modules. For instance:
*   The Gibbs energy minimizer (`liboceqplus`) calls `noel()`, `noph()`, `get_phase_data(...)`, `set_constitution(...)`, etc., during its iterative calculations.
*   The TQ interface layer (`liboctq`) uses functions like `find_phase_by_name(...)`, `get_element_data(...)`, and `get_phase_name(...)` to respond to user queries or commands.
*   Database reading routines (e.g., in `gtp3E.F90`) use these functions to populate the GTP data structures with information from TDB files.

# Dependencies and Interactions

*   **Core Data Structures (from `gtp3_dd1.F90`, `gtp3_dd2.F90`):** The routines in `gtp3A.F90` operate directly on the fundamental data structures (derived types and arrays) that define the thermodynamic system. These data structures are declared in other files included by `gtp3.F90`.
*   **Other GTP Components:** `gtp3A.F90` functions are called by, and call functions within, other `gtp3*.F90` files that make up the `GENERAL_THERMODYNAMIC_PACKAGE`. For example, `set_reference_state` might implicitly use calculation routines from `gtp3X.F90` to determine phase stability.
*   **Higher-Level Modules:** This file provides the API for higher-level modules like the Gibbs energy minimizer (`liboceqplus`) and various user interface layers to access and manipulate thermodynamic information.

This file is essential for the basic setup, query, and data management aspects of the General Thermodynamic Package.
