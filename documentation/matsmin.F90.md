# Overview

The Fortran module `liboceqplus` (found in `src/minimizer/matsmin.F90`) implements Hillert's Minimizer as adapted by Sundman (HMS). This module is a cornerstone of the OpenCalphad software, responsible for performing the core Gibbs energy minimization to determine phase equilibria under specified conditions. The implementation is based on the principles outlined in Mats Hillert's 1981 paper in Physica and Bo Jansson's 1984 thesis, with further details on this specific implementation published by Sundman in Computational Materials Science (2015).

The minimizer iteratively adjusts phase constitutions and amounts to find the state of minimum Gibbs energy for the system, considering various thermodynamic models and user-defined conditions.

# Key Components

## Module `liboceqplus`

*   **Description:** This module encapsulates the entire Hillert-Mats-Sundman (HMS) Gibbs energy minimizer, including derived types for managing calculation state and numerous subroutines for different stages of the equilibrium calculation.
*   **Version:** `hmsversion` (e.g., `'HMS-3.0'`)

## Derived Types

*   **`meq_phase`**
    *   **Description:** Stores phase-specific data and results during an equilibrium calculation. This includes pointers to underlying phase data, calculated mole fractions, the inverted phase matrix, and status flags.
    *   **Key Members:** `iph` (phase index), `ics` (composition set index), `stable` (flag indicating if phase is in the stable set), `invmat` (inverted phase matrix for constituent calculations), `xmol` (mole fractions of components in the phase), `curd` (pointer to `gtp_phase_varres` from `general_thermodynamic_package`).

*   **`meq_setup`**
    *   **Description:** Holds global data and setup information for a single equilibrium calculation instance. It tracks the overall state of the minimization process.
    *   **Key Members:** `nphase` (total number of phases considered), `nstph` (current number of stable phases), `nrel` (number of system components/elements), `iphl`, `icsl`, `aphl` (initial guess for stable phases and their amounts), `stphl` (list of currently stable phases), `phr` (array of `meq_phase` type, one for each active phase), `status` (control bits like `MMQUIET`, `MMNOSTARTVAL`).

*   **`map_fixph`**
    *   **Description:** Used specifically for mapping calculations (e.g., phase diagrams). It stores information about fixed and stable phase sets across different points in a map.
    *   **Key Members:** `nfixph` (number of fixed phases), `nstabph` (number of stable phases), `fixph` (array of `gtp_phasetuple` for fixed phases), `stableph` (array of `gtp_phasetuple` for stable phases).

## Core Equilibrium Calculation Subroutines

*   **`calceq2(mode, ceq)` / `calceq3(mode, confirm, ceq)`**
    *   **Description:** High-level entry points for initiating an equilibrium calculation. They set up the `meq_setup` record and call `calceq7`. `calceq2` includes timing and more verbose output by default, while `calceq3`'s output is controlled by `confirm`.
    *   **Parameters:** `mode` (Integer, controlling calculation strategy, e.g., use of grid minimizer), `ceq` (Pointer to `gtp_equilibrium_data`), `confirm` (Logical, for `calceq3` output).

*   **`calceq7(mode, meqrec, mapfix, ceq)`**
    *   **Description:** A central routine that orchestrates the equilibrium calculation. It handles different modes (full calculation, step/map update), extracts conditions from `ceq`, optionally calls a global grid minimizer (`global_gridmin`) for an initial guess, and then invokes `meq_phaseset` to perform the iterative minimization.
    *   **Parameters:** `mode` (Integer), `meqrec` (Pointer to `meq_setup`), `mapfix` (Allocatable `map_fixph`, for mapping), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`meq_phaseset(meqrec, formap, mapfix, ceq)`**
    *   **Description:** Manages the set of stable phases. It iteratively calls `meq_sameset` and adjusts the set of phases considered stable based on their driving forces or if they become unstable, until a consistent and converged phase set is found.
    *   **Parameters:** `meqrec` (`meq_setup` instance), `formap` (Logical, true if called during mapping), `mapfix` (Allocatable `map_fixph`), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`meq_sameset(irem, iadd, mapx, meqrec, phr, inmap, ceq)` (Recursive)**
    *   **Description:** Performs iterations for a *fixed* set of stable phases. It sets up the equilibrium equations using `setup_equilmatrix`, solves them (typically using `lingld`) to get chemical potentials and phase amounts, then calls `meq_onephase` for each phase to update its internal constitution (site fractions). It checks for convergence and determines if any phases should be added (`iadd`) or removed (`irem`) from the stable set.
    *   **Parameters:** `irem` (Index of phase to remove, OUT), `iadd` (Index of phase to add, OUT), `mapx` (Integer, for mapping context), `meqrec` (`meq_setup` instance), `phr` (Array of `meq_phase`), `inmap` (Integer, mapping flag), `ceq` (Pointer to `gtp_equilibrium_data`).

## Phase-Specific and Matrix Setup Subroutines

*   **`meq_onephase(meqrec, pmi, ceq)`**
    *   **Description:** Calculates properties for a single phase (`pmi` points to a `meq_phase` instance). This includes its Gibbs energy, derivatives with respect to constituent fractions, and the construction and inversion of its phase matrix (relating constituent chemical potentials to site fractions). It handles different thermodynamic models (ideal, stoichiometric, CEF, ionic liquids).
    *   **Parameters:** `meqrec` (`meq_setup` instance), `pmi` (Pointer to `meq_phase`), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`setup_equilmatrix(meqrec, phr, nz1, smat, tcol, pcol, dncol, converged, ceq)`**
    *   **Description:** Constructs the main system of linear equations (the "equilibrium matrix", `smat`) that needs to be solved to find the chemical potentials of components and amounts of stable phases. It incorporates user-defined conditions (e.g., fixed temperature, pressure, composition, chemical potentials, phase amounts).
    *   **Parameters:** `meqrec` (`meq_setup` instance), `phr` (Array of `meq_phase`), `nz1` (Dimension of matrix), `smat` (The matrix, OUT), `tcol`, `pcol` (Indices for T, P columns if variable), `dncol` (Index for phase amount columns), `converged` (Status flag), `ceq` (Pointer to `gtp_equilibrium_data`).

## Other Notable Subroutines

*   **`equilph1a` through `equilph1e`**: A set of routines likely for more specialized single-phase or specific equilibrium calculations, possibly related to diffusion data (`equilph1d`, `equilph1e` take `mugrad` and `mobval` arguments).
*   **`tzero(iph1, iph2, icond, value, ceq)`**: Calculates the value of a condition `icond` for which two phases `iph1` and `iph2` have the same Gibbs energy (T0 temperature).
*   **`liquid_eet(iph1, iph2, icond, value, ceq)`**: Calculates the Equal Entropy Temperature (EET) for two liquid phases.
*   **`calc_paraeq(...)`**: Calculates paraequilibrium conditions.
*   **`calc_dgdyterms1X`, `calc_dgdytermshm`**: Subroutines involved in computing terms for the derivatives of Gibbs energy with respect to constituent fractions, which are essential for constructing the equilibrium matrix and for calculating thermodynamic factors.
*   **`assessment_calfun`**: Called by MINPACK routines during parameter optimization (assessment). It calculates the error (difference between experimental and calculated values) for a given set of model parameters.
*   **`meq_evaluate_all_svfun`, `meq_get_state_varorfun_value`, `meq_evaluate_svfun`, `meq_state_var_dot_derivative`**: Routines for evaluating state variables and their derivatives, including complex user-defined functions.

# Important Variables/Constants

*   **`hmsversion` (Character, Parameter):** Version string for the HMS minimizer (e.g., `'HMS-3.0'`).
*   **`MMQUIET`, `MMNOSTARTVAL`, `MMSTEPINV` (Integer, Parameters):** Bit flags used in `meqrec%status` to control minimizer behavior:
    *   `MMQUIET`: Suppress output during calculation.
    *   `MMNOSTARTVAL`: Do not call the global grid minimizer for initial start values.
    *   `MMSTEPINV`: Related to invariant steps in mapping.
*   **`mmdebug` (Integer, Module Variable):** Debug flag.
*   **`mmdotder` (Integer, Module Variable):** Flag to indicate if a dot derivative (like Cp) calculation is in progress, which might affect how phase data is handled.
*   **Default convergence parameters (e.g., `default_minadd`, `default_minrem`, `default_deltaT`):** These are likely defined in `general_thermodynamic_package` or `ocparam.F90` but are used by `matsmin.F90` to control iteration behavior and convergence criteria.

# Usage Examples

The subroutines within `liboceqplus` are generally not called directly by end-users. They form the core computational engine for thermodynamic equilibrium calculations. Higher-level interfaces, such as:
*   The command-line interface of OpenCalphad.
*   The TQ interface (`liboctq.F90`).
*   Python wrappers (`pyOC.py` via `pyOC.f90`).

These interfaces translate user requests (e.g., "calculate equilibrium at 1000K for Fe-C") into a series of calls to set up conditions within the `gtp_equilibrium_data` structure, and then invoke routines like `calceq2` or `calceq3`.

For example, a call to `pyOC.py`'s `OpenCalphad.calculateEquilibrium()` method would ultimately lead to `calceq2` or `calceq3` being executed within the Fortran backend.

# Dependencies and Interactions

*   **`general_thermodynamic_package` (Fortran Module):** This is a crucial dependency. It provides fundamental thermodynamic models, definitions for data structures like `gtp_equilibrium_data`, `gtp_phase_varres`, `gtp_phasetuple`, `gtp_condition`, and routines for accessing and manipulating thermodynamic data (e.g., `get_phase_data`, `set_constitution`, `calcg`).
*   **`minpack` (Fortran Module/Library):** The `use minpack` statement indicates a dependency on the MINPACK library, which is a collection of Fortran subprograms for the numerical solution of systems of nonlinear equations and nonlinear least squares problems. This is likely used by the `assessment_calfun` routine for parameter optimization. The `hybrd1` routine from MINPACK is explicitly used in `tzero` and `calc_paraeq`.
*   **Internal Structure:** The module is highly interconnected. `calceq2`/`calceq3` call `calceq7`, which calls `meq_phaseset`, which recursively calls `meq_sameset`. `meq_sameset` then calls `setup_equilmatrix` and `meq_onephase`.
*   **Role in OpenCalphad:** `liboceqplus` is the heart of the equilibrium calculation capabilities of OpenCalphad. It takes system descriptions and conditions and determines the stable phase assemblage and their compositions.
