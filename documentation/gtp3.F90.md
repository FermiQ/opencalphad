# Overview

The Fortran file `src/models/gtp3.F90` is the central aggregator for the **General Thermodynamic Package (GTP)** within the OpenCalphad software. It defines the Fortran module `GENERAL_THERMODYNAMIC_PACKAGE`. This module is the core thermodynamic engine of OpenCalphad, responsible for handling the storage, management, and calculation of thermodynamic data and models for various phases and components. It provides the foundational routines and data structures necessary for thermodynamic modeling, property evaluation, and equilibrium calculations performed by other parts of the system, such as the Gibbs energy minimizer.

Instead of containing all the code directly, `gtp3.F90` primarily uses `#include` preprocessor directives to assemble the complete module from a collection of specialized `gtp3*.F90` files.

# Key Components

## Module `GENERAL_THERMODYNAMIC_PACKAGE`

*   **Description:** This module serves as the comprehensive thermodynamic library for OpenCalphad. It encompasses data structures for representing elements, species, phases, thermodynamic models, and system conditions, along with subroutines and functions to perform a wide range of thermodynamic calculations.
*   **Version:** The module declares a `version` parameter (e.g., `'  6.084 '`) representing the overall OpenCalphad software version.

## Included Source Files

The functionality of `GENERAL_THERMODYNAMIC_PACKAGE` is built by including several other Fortran files. Each included file likely contributes a specific set of capabilities:

*   **`gtp3_dd1.F90`**: Likely handles data structures and basic operations for Temperature-Pressure (TP) functions, especially for systems not relying on encrypted databases.
*   **`gtp3_dd2.F90`**: Probably defines the core and more complex data structures used throughout GTP, such as those for elements, species, phases (including constituent arrays, sublattices), model parameters, equilibrium states, and conditions.
*   **`gtp3_xml.F90`**: Suggests functionalities related to parsing and generating XML data, which could be used for reading thermodynamic databases in XML format (e.g., certain TDB formats or future XML-based standards) or for data exchange.
*   **`gtp3A.F90`**: Comments indicate "initialization, how many, find things, get things, set things." This file likely provides routines for:
    *   Initializing the thermodynamic system and its data structures.
    *   Querying the number of defined entities (e.g., elements, phases, species).
    *   Searching for and retrieving specific thermodynamic entities or parameters.
    *   Setting or modifying properties of these entities.
*   **`gtp3B.F90`**: Comments indicate "enter data." This file probably contains subroutines that allow users or other program parts to input new thermodynamic data, such as model parameters, phase information, or component definitions, into the GTP system.
*   **`gtp3C.F90`**: Comments indicate "list data." This file likely includes routines for displaying or printing various thermodynamic data, model parameters, and system information stored within GTP.
*   **`gtp3D.F90`**: Comments indicate "save and read from files." This suggests general-purpose file input/output capabilities for saving and restoring the state of the thermodynamic system or parts of it, possibly in internal formats.
*   **`gtp3E.F90`**: Comments indicate "Read/write TDB/UNFORMATTED." This file specializes in handling Thermo-Calc Database (TDB) files, a common format for storing thermodynamic data, and possibly OpenCalphad's own unformatted binary files for faster data access.
*   **`gtp3EX.F90`, `gtp3EY.F90`**: Comments indicate "Read/write XML." These files likely provide more extensive or specific XML handling capabilities, perhaps for different XML schemas or advanced XML operations related to thermodynamic data.
*   **`gtp3F.F90`**: Comments indicate "state variable functions, interactive things." This file probably contains routines for defining, managing, and evaluating complex, user-defined state variable functions (e.g., functions of T, P, composition). It might also include components for interactive command processing.
*   **`gtp3G.F90`**: Comments indicate "status for things, unfinished things, internal stuff." This is likely a collection of utility routines for managing status flags, handling features that are under development, or providing internal helper functions used by other parts of GTP.
*   **`gtp3H.F90`**: Comments indicate "Additions (magnetic and others)." This file probably implements additional physical models beyond basic solution models, such as contributions from magnetic ordering, order-disorder transitions, or other specialized thermodynamic effects.
*   **`gtp3X.F90`, `gtp3XQ.F90`**: Comments indicate "calculate things, gtp3XQ for MQMQA." These files contain core routines for calculating thermodynamic properties (Gibbs energy, enthalpy, entropy, derivatives) based on the active models. `gtp3XQ.F90` is specifically mentioned for the Modified Quasichemical Model in the Quadruplet Approximation.
*   **`gtp3Y.F90`**: Comments indicate "Grid minimizer and miscellaneous." This file likely houses the global grid minimization algorithm used for initial phase stability assessment by sampling Gibbs energies across composition space, and possibly other miscellaneous utility functions.
*   **`gtp3Z.F90`**: Comments indicate "TPFUN routines for non-encrypted databases." This file likely provides further, more specialized routines for handling Temperature-Pressure functions, particularly for systems where data is not encrypted.

# Important Variables/Constants

*   **`version` (Character, Parameter):** Stores the overall OpenCalphad version string (e.g., `'  6.084 '`). This is defined within the `GENERAL_THERMODYNAMIC_PACKAGE` module.

# Usage Examples

The `GENERAL_THERMODYNAMIC_PACKAGE` module is a foundational library within OpenCalphad. It is not typically used directly by an end-user writing Fortran code from scratch. Instead, its subroutines and functions are the workhorses called by other higher-level components of the OpenCalphad system.

For instance:
*   When a user issues a command to calculate an equilibrium, the Gibbs energy minimizer (e.g., `liboceqplus` from `matsmin.F90`) will make numerous calls to routines in `GENERAL_THERMODYNAMIC_PACKAGE` to:
    *   Retrieve phase definitions and model parameters.
    *   Calculate Gibbs energies and their derivatives for various phases at different compositions, temperatures, and pressures.
    *   Access component data and system conditions.
*   When a TDB file is read, routines from `gtp3E.F90` (included in this package) are used to parse the file and populate the internal GTP data structures.
*   Python or other high-level interfaces ultimately rely on this package to perform the underlying thermodynamic computations.

# Dependencies and Interactions

*   **`ocnum` (Fortran Module):** Provides numerical utility functions that might be used by GTP for various calculations.
*   **`metlib` (Fortran Module):** Likely a general utility library for OpenCalphad, potentially providing string manipulation, error handling, or other common functions.
*   **`ocparam` (Fortran Module):** Defines global parameters, constants, and possibly default settings used throughout OpenCalphad, including within GTP.
*   **Included `gtp3*.F90` Files:** As detailed above, these files are the primary components that are assembled by `gtp3.F90` to form the complete `GENERAL_THERMODYNAMIC_PACKAGE`. The main `gtp3.F90` file acts as an aggregator and defines the module scope for these included parts.
*   **OpenCalphad Minimizer (`liboceqplus`):** The GTP module is critically linked with the Gibbs energy minimizer. The minimizer uses GTP routines to get all necessary thermodynamic information (Gibbs energies, activities, phase compositions, derivatives) to find the equilibrium state.
*   **Database Interfaces:** GTP interacts with routines responsible for reading and interpreting thermodynamic databases (e.g., TDB files, XML files).
*   **User Interfaces (Command Line, Python API):** Higher-level user interfaces utilize GTP indirectly to set up calculations, retrieve data, and display results.

In essence, `GENERAL_THERMODYNAMIC_PACKAGE` serves as the central nervous system for all thermodynamic modeling and calculations within OpenCalphad.
