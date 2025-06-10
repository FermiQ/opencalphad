# Overview

The Fortran file `src/models/gtp3D.F90` is a part of the General Thermodynamic Package (GTP) in OpenCalphad, included within the main `GENERAL_THERMODYNAMIC_PACKAGE` module. Contrary to earlier high-level comments suggesting general file saving/reading, this file's content (and more detailed comments in `gtp3A.F90` which categorize it as "8. Interactive things") primarily focuses on providing subroutines for interactive data entry, amendment of existing data, and the complex parsing and application of system conditions. It serves as an interface layer between user-provided command strings (or programmatic calls) and the underlying GTP data structures and calculation routines.

# Key Components

This file contains routines for interactive data manipulation and condition handling:

## Interactive Data Entry and Amendment

*   **`ask_phase_constitution(cline, last, iph, ics, lokcs, ceq)`**
    *   **Description:** Interactively prompts the user to modify the constitution (site fractions) of a specified phase (`iph`, `ics`). It allows setting the phase amount and choosing to use the current constitution, a default constitution, or enter new fractions constituent by constituent.
    *   **Parameters/Arguments:** `cline` (Input command string), `last` (Position in `cline`), `iph`, `ics`, `lokcs` (Phase/composition set identifiers), `ceq` (Pointer to equilibrium data).

*   **`ask_phase_new_constitution(cline, last, iph, ics, lokcs, ceq)`**
    *   **Description:** Similar to `ask_phase_constitution`, this routine also allows interactive input of phase constitution. The exact difference might be in prompting or default behaviors.

*   **`enter_parameter_interactivly(cline, ip, mode)`**
    *   **Description:** Provides an interactive way to enter or amend a thermodynamic model parameter. It parses a parameter name (e.g., "G(SIGMA,FE:CR:FE,CR;1)"), then prompts for the T-dependent function expression and a bibliographic reference.
    *   **Parameters/Arguments:** `cline` (Input command string), `ip` (Position in `cline`), `mode` (0 for entering, 1 for listing).

*   **`amend_global_data(cline, ipos)`**
    *   **Description:** Allows interactive modification of global OpenCalphad settings. This includes the system name, user expertise level (Beginner, Frequent, Expert), and flags controlling the behavior of the grid minimizer (e.g., if global minimization is allowed, if merging of grid points is permitted, if automatic creation/deletion of composition sets is enabled).
    *   **Parameters/Arguments:** `cline` (Input command string), `ipos` (Position in `cline`).

*   **`enter_bibliography_interactivly(cline, last, mode, iref)`**
    *   **Description:** Interactively enters or amends a bibliographic reference string associated with a reference identifier.
    *   **Parameters/Arguments:** `cline` (Input command string), `last` (Position in `cline`), `mode` (0 for enter, 1 for amend), `iref` (Reference index).

*   **`set_input_amounts(cline, lpos, ceq)`**
    *   **Description:** Allows users to specify input amounts of various species, either in moles (`N(species)=value`) or mass (`B(species)=value`). These are then converted into elemental mole conditions for the equilibrium calculation.
    *   **Parameters/Arguments:** `cline` (Input command string), `lpos` (Position in `cline`), `ceq` (Pointer to equilibrium data).

## Condition and Experiment Handling

*   **`enter_experiment(cline, ip, ceq)`**
    *   **Description:** Parses a command string to define an experimental data point. This includes the experimental value and its uncertainty (which can be a numerical value or a symbolic reference to a constant). It uses `set_cond_or_exp` internally.
    *   **Parameters/Arguments:** `cline` (Input command string), `ip` (Position in `cline`), `ceq` (Pointer to equilibrium data).

*   **`set_condition(cline, ip, ceq)`**
    *   **Description:** A wrapper routine that calls `set_cond_or_exp` to parse and establish a thermodynamic condition from a command string.
    *   **Parameters/Arguments:** `cline` (Input command string), `ip` (Position in `cline`), `ceq` (Pointer to equilibrium data).

*   **`set_cond_or_exp(cline, ip, new, notcond, ceq)`**
    *   **Description:** A core parsing routine for both conditions (`notcond=0`) and experiments (`notcond=1`). It decodes a command string that can define complex conditions or experiments, potentially involving expressions with multiple state variable terms, coefficients, and values (numerical or symbolic). It creates or updates `gtp_condition` records in the specified equilibrium `ceq`. It handles operators like `=`, `>`, `<`, `:=`.
    *   **Parameters/Arguments:** `cline` (Input command string), `ip` (Position in `cline`), `new` (Pointer to the created/modified condition record), `notcond` (Flag: 0 for condition, 1 for experiment), `ceq` (Pointer to equilibrium data).

*   **`apply_condition_value(current, what, value, cmix, ccf, ceq)`**
    *   **Description:** This crucial subroutine is used by the equilibrium minimizer (e.g., `setup_equilmatrix` in `matsmin.F90`).
        *   If `what < 0`: It analyzes the type of condition pointed to by `current` and returns classification information in the `cmix` array (e.g., if it's a condition on T, P, chemical potential, fixed phase amount, or other extensive properties).
        *   If `what = 0`: It calculates the current `value` of the state variable expression defined by the condition `current`, using the present state of the equilibrium `ceq`.
    *   **Parameters/Arguments:** `current` (Pointer to a `gtp_condition`), `what` (Mode of operation), `value` (Calculated value, OUT), `cmix` (Array for classification, OUT), `ccf` (Coefficients for multi-term conditions, OUT), `ceq` (Pointer to equilibrium data).

*   **`get_experiment_with_symbol(symsym, experimenttype, temp)`**: Finds an existing experiment record based on a symbol index and experiment type.
*   **`get_condition_expression(nterm, svrarr, pcond)`**: Finds a condition record matching a multi-term state variable expression.
*   **`get_condition(nterm, svr, pcond)`**: Finds a specific condition record. If `nterm < 0`, it finds the `-nterm`-th active condition. Otherwise, it searches for a condition matching a single state variable `svr`.
*   **`locate_condition(seqz, pcond, ceq)`**: Finds a condition record by its sequential number `seqz`.
*   **`condition_value(mode, pcond, value, ceq)`**: Sets (if `mode=0`) or gets (if `mode=1`) the numerical `value` of a condition pointed to by `pcond`.

# Important Variables/Constants

This file primarily uses variables passed as arguments or accessed from the main GTP data structures defined in other included files. There are no major module-level constants specific to this file's direct functions, beyond what's used from `ocparam` or other GTP parts.

# Usage Examples

The routines in `gtp3D.F90` are typically invoked by:
*   **OpenCalphad's Command Line Parser:** When users enter commands such as `AMEND GLOBAL-DATA`, `SET_CONDITION T=1000 P=1E5 X(FE)=0.9...`, `ENTER_EXPERIMENT ...`, `AMEND PARAMETER ...`, or `SET_CONSTITUTION ...`.
*   **Equilibrium Minimizer (`matsmin.F90`):** The `apply_condition_value` subroutine is a critical internal function called repeatedly by the minimizer to evaluate how well the current state of the system satisfies the defined thermodynamic conditions.
*   **Scripting Interfaces:** High-level scripting interfaces might use these routines to programmatically define or modify systems and experiments.

# Dependencies and Interactions

*   **Core GTP Data Structures:** The subroutines heavily rely on and modify the core GTP data structures for elements, species, phases, conditions, and equilibria (defined in `gtp3_dd1.F90`, `gtp3_dd2.F90`).
*   **GTP Utilities:** Uses various helper functions from other GTP components for tasks like finding phases (`find_phase_by_name`), decoding state variables (`decode_state_variable`), encoding state variables (`encode_state_variable`), setting constitution (`set_constitution`), and managing TP functions.
*   **Metlib Utilities (`metlib`):** Employs parsing utilities like `gparcx` (get parameter character string), `gparrdx` (get parameter real double), `getrel` (get real), `getint` (get integer) for processing command line input.
*   **Equilibrium Minimizer (`liboceqplus`):** The `apply_condition_value` routine is a key service provider to the minimizer, enabling it to check constraints during the Gibbs energy minimization process.
*   **File I/O (Limited):** While the initial high-level comment for `gtp3.F90` suggested "save and read from files" for `gtp3D.F90`, its actual content is more geared towards interactive commands and condition processing. General database file I/O is more concentrated in `gtp3E.F90` (TDB) and `gtp3EX.F90`/`gtp3EY.F90` (XML).

This file plays a significant role in user interaction, system definition, and the application of constraints within thermodynamic calculations.
