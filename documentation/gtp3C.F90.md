# Overview

The Fortran file `src/models/gtp3C.F90` is a component of the General Thermodynamic Package (GTP) in OpenCalphad, included within the main `GENERAL_THERMODYNAMIC_PACKAGE` module. As suggested by comments in `gtp3.F90` ("10: list data"), this file provides a collection of subroutines dedicated to listing, displaying, and formatting various thermodynamic data and calculation results stored within the GTP system. These routines are essential for user interaction, debugging, and generating output in human-readable or specific file formats (like TDB).

# Key Components

This file contains a variety of subroutines for outputting different kinds of information:

## Data Listing Subroutines

*   **`list_all_elements(unit)` / `list_all_elements2(unit)`**
    *   **Description:** Lists all defined elements in the system to the specified Fortran `unit`. `list_all_elements` provides a formatted table with details like symbol, name, reference state, mass, H298-H0, S298, and status. `list_all_elements2` outputs in a format closer to TDB ELEMENT commands.
    *   **Parameters/Arguments:** `unit` (Integer, output unit).

*   **`list_all_components(unit, ceq)`**
    *   **Description:** Lists the active system components for a given equilibrium `ceq`. Output includes component number, symbol, moles, mass percent, chemical potential, and reference state.
    *   **Parameters/Arguments:** `unit` (Integer, output unit), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_all_species(unit)`**
    *   **Description:** Lists all defined species, including their symbol, stoichiometry, mass, charge, and status flags. Also lists UNIQUAC parameters if applicable.
    *   **Parameters/Arguments:** `unit` (Integer, output unit).

*   **`list_phase_model(iph, ics, lut, CHTD, ceq)`**
    *   **Description:** Displays the model definition for a specific phase `iph` and composition set `ics`. This includes the phase name, model name, number of sublattices, site occupancies, and a list of constituents in each sublattice.
    *   **Parameters/Arguments:** `iph` (Integer, phase index), `ics` (Integer, composition set index), `lut` (Integer, output unit), `CHTD` (Character, for TDB type definition character), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_phase_data(iph, CHTD, lut)`**
    *   **Description:** Lists the thermodynamic parameters (endmembers and interactions) for a given phase `iph`. It iterates through the phase's parameter records and uses `list_tpfun` to display the T-dependent expression for each parameter. It handles disordered fraction sets and Toop/Kohler ternary extrapolations.
    *   **Parameters/Arguments:** `iph` (Integer, phase index), `CHTD` (Character, for TDB type definition character), `lut` (Integer, output unit).

*   **`list_phase_data2(iph, ftyp, CHTD, lut)`**
    *   **Description:** Used by `list_TDB_format` (and thus `SAVE TDB` command) to output phase definitions and parameters in TDB format.
    *   **Parameters/Arguments:** `iph` (Integer, phase index), `ftyp` (Integer, format type, 2 for TDB), `CHTD` (Character, TDB type definition character), `lut` (Integer, output unit).

*   **`list_bibliography(bibid, lut)`**
    *   **Description:** Lists bibliographic references stored in the system. Can list all or a specific reference `bibid`.
    *   **Parameters/Arguments:** `bibid` (Character, reference ID or blank for all), `lut` (Integer, output unit).

*   **`list_defined_properties(lut)`**
    *   **Description:** Lists all model parameter identifiers (e.g., G, TC, BMAG, MQ, V0) known to the system, along with their specifications (T/P dependence, association with elements/constituents).
    *   **Parameters/Arguments:** `lut` (Integer, output unit).

## Equilibrium and Condition Listing Subroutines

*   **`list_conditions(lut, ceq)`**
    *   **Description:** Lists all currently active and inactive conditions for a given equilibrium `ceq`. Uses `get_all_conditions` to format the output.
    *   **Parameters/Arguments:** `lut` (Integer, output unit), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_global_results(lut, ceq)`**
    *   **Description:** Lists global results for a calculated equilibrium `ceq`, such as Temperature (K and C), Pressure (Pa), Volume (m³), total moles (N), total mass (B), Gas Constant (R*T), Gibbs energy (G, G/N), Enthalpy (H), and Entropy (S). Values are typically referred to SER.
    *   **Parameters/Arguments:** `lut` (Integer, output unit), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_components_result(lut, mode, ceq)`**
    *   **Description:** Lists results on a per-component basis for equilibrium `ceq`. Output includes component name, moles, mole or mass fraction (controlled by `mode`), chemical potential (as µ/RT), activity, and reference state.
    *   **Parameters/Arguments:** `lut` (Integer, output unit), `mode` (Integer, 1 for mole_fr, 2 for mass_fr), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_phase_results(iph, jcs, mode, lut, once, ceq)`**
    *   **Description:** Provides a detailed listing of results for a specific phase `iph` and composition set `jcs`. The `mode` argument controls various aspects:
        *   Units digit: Mole fraction (0) or mass fraction (otherwise).
        *   Tens digit: Composition only (0) or include constitution (1).
        *   Hundreds digit: Value order (0) or alphabetical order (1) for constituents/components.
        *   Thousands digit: List all phases (0) or only stable phases (1).
        *   Ten thousands digit: If set (1), lists constituent fractions multiplied by formula units (Solgasmix style).
        Output includes phase name, status, molar amount, atoms/FU, driving force (dGm/RT), and then composition and optionally constitution.
    *   **Parameters/Arguments:** As listed above. `once` is a logical for controlling header printing in loops.

*   **`list_sorted_phases(unit, mode, ceq)`**
    *   **Description:** Lists phases sorted by their stability status and driving force: stable phases first (ordered by amount), then entered-but-unstable phases (ordered by driving force), then dormant phases (ordered by driving force). Suspended phases are listed separately.
    *   **Parameters/Arguments:** `unit` (Integer, output unit), `mode` (Integer, controls if status bits are displayed), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_all_phases(unit, ceq)`**
    *   **Description:** Lists all phases defined in the system for a given equilibrium `ceq`, showing their name, composition set info, moles, atoms/FU, driving force, and status bits.
    *   **Parameters/Arguments:** `unit` (Integer, output unit), `ceq` (Pointer to `gtp_equilibrium_data`).

*   **`list_phases_with_positive_dgm(mode, lut, ceq)`**
    *   **Description:** Specifically lists dormant or entered phases that have a positive driving force, indicating they would like to become stable.
    *   **Parameters/Arguments:** `mode` (Integer, unused), `lut` (Integer, output unit), `ceq` (Pointer to `gtp_equilibrium_data`).

## Formatting and Helper Subroutines

*   **`list_element_data(text, ipos, elno)`**: Formats data for a single element into a character string `text`.
*   **`list_species_data(text, ipos, spno)` / `list_species_data2(text, ipos, loksp)`**: Formats data for a single species into `text`.
*   **`format_phase_composition(mode, nv, consts, vals, lut)`**: Helper routine to format and list phase compositions or constitutions in multiple columns, with options for alphabetical or value order.
*   **`get_one_condition(ip, text, seqz, ceq)` / `get_all_conditions(text, mode, ceq)`**: Retrieve and format one or all conditions for an equilibrium into a text string.
*   **`get_one_experiment(ip, text, seqz, eval, ceq)`**: Retrieve and format one experiment description into a text string, optionally evaluating the current calculated value.
*   **`encode_stoik(text, ipos, mdig, spno)`**: Encodes the stoichiometry of a species `spno` into a human-readable string like "FE2O3". (Counterpart `decode_stoik` is in `gtp3B.F90`).

## High-Level Output Commands

*   **`list_many_formats(cline, last, ftyp, unit1)`**: A command-level routine to output the entire database in various formats. `ftyp` determines the format: 1 for Screen, 2 for TDB (calls `list_TDB_format`), 3 for Macro (not implemented), 4 for LaTeX (not implemented), 6 for XTDB (uses SAVE command).
*   **`list_TDB_format(filename)`**: Outputs the entire thermodynamic database (elements, species, functions, phases, parameters, references) to the specified `filename` in TDB format.

# Important Variables/Constants

*   **`kou` (Integer, from `ocparam`):** Likely the default Fortran unit number for standard output, used frequently in these listing routines when `unit` or `lut` is set to `kou`.
*   **Formatting Strings:** The subroutines contain numerous Fortran `FORMAT` statements that define the layout of the output.
*   **Status Flags (e.g., `PHSUS`, `PHDORM`, `PHFIXED`, `PHENTSTAB`):** These named constants (likely defined in `gtp3_dd2.F90` or `ocparam`) are used to interpret and display phase statuses.

# Usage Examples

The routines in `gtp3C.F90` are primarily called by:
*   **User Interface Commands:** When a user issues a command like `LIST_ELEMENTS`, `DISPLAY_PHASE FCC_A1`, `LIST_EQUILIBRIUM`, or `SAVE TDB mydata.tdb` in the OpenCalphad interactive environment or a script.
*   **Debugging Code:** Developers might insert calls to these listing routines to inspect the state of GTP data structures at various points during execution.
*   **Higher-Level API Functions:** Python wrappers or other APIs would use these routines to present data to the end-user.

For example, after an equilibrium calculation using the minimizer (`matsmin.F90`), routines like `list_global_results`, `list_components_result`, and `list_sorted_phases` (or `list_phase_results`) would be called to display the outcome of the calculation.

# Dependencies and Interactions

*   **GTP Core Data Structures:** All listing routines extensively read from the core GTP data structures (e.g., `ellista`, `splista`, `phlista`, `phasetuple`, `eqlista`, `svflista`, `propid`, `bibrefs`, and the contents of `gtp_equilibrium_data` like `phase_varres`) which are defined in `gtp3_dd1.F90` and `gtp3_dd2.F90`.
*   **Utility Functions:** They use various helper functions from other GTP parts (e.g., `get_phase_name`, `get_element_data`, `encode_stoik` from `gtp3A.F90` or `gtp3B.F90`) and general utilities from `metlib` (e.g., `wriint`, `wrice2`, `capson`).
*   **Output Units:** They interact with Fortran I/O units to send formatted text to the console or files.
*   **Error Handling:** They check and sometimes reset the global error flag `gx%bmperr`.

This file is crucial for providing visibility into the thermodynamic database and the results of calculations within the OpenCalphad system.
