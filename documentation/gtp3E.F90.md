# Overview

The Fortran file `src/models/gtp3E.F90` is a component of the General Thermodynamic Package (GTP) in OpenCalphad, included within the `GENERAL_THERMODYNAMIC_PACKAGE` module. Its primary responsibility is to manage the reading of thermodynamic data from TDB (Thermo-Calc Database) files and to handle the saving and loading of the entire GTP workspace to/from platform-specific unformatted binary files. This allows for persistent storage of thermodynamic databases and session states.

# Key Components

## TDB File Reading

*   **`readtdb(filename, nel, selel)`**
    *   **Description:** This is the main subroutine for parsing and processing a TDB file. It reads the file line by line, identifies TDB commands (e.g., `ELEMENT`, `SPECIES`, `PHASE`, `CONSTITUENT`, `FUNCTION`, `PARAMETER`, `TYPE_DEFINITION`), and extracts the associated data. It then calls various data entry routines (primarily from `gtp3B.F90`) to populate the internal GTP data structures. It supports selective reading of elements based on the `selel` array.
    *   **Parameters/Arguments:** `filename` (Character, path to TDB file), `nel` (Integer, number of selected elements; 0 to read all), `selel` (Character array, list of selected element symbols).

*   **`readtdbsilent()`**
    *   **Description:** A utility subroutine that sets a global flag (`GSSILENT`) to suppress most console output during the TDB reading process.

*   **`any_disordered_part(lin, ndisph, dispartph, ordpartph, orddistyp)`**
    *   **Description:** Scans a TDB file (opened on unit `lin`) specifically to identify phases defined with disordered parts using `TYPE_DEFINITION ... DIS_PART` commands. It stores the names of the ordered and disordered parts for later processing during the main TDB read.
    *   **Parameters/Arguments:** `lin` (Integer, file unit), `ndisph` (Integer, number of disordered phases found, OUT), `dispartph` (Character array, names of disordered phase parts, OUT), `ordpartph` (Character array, names of corresponding ordered phase parts, OUT), `orddistyp` (Integer array, type of disordering, OUT).

*   **`checkdb2(filename, ext, nel, selel)`**
    *   **Description:** Checks if a TDB or XTDB file specified by `filename` (with default extension `ext`) exists. It then scans the file to extract a list of all unique element symbols present (`selel`) and their count (`nel`). It also displays any `DATABASE_INFORMATION` sections found in the TDB file.
    *   **Parameters/Arguments:** `filename` (Character, file path), `ext` (Character, default extension, e.g., ".TDB"), `nel` (Integer, number of elements found, OUT), `selel` (Character array, list of element symbols found, OUT).

*   **`istdbkeyword(text, nextc) RESULT(integer)`**
    *   **Description:** A utility function that checks if the beginning of the input `text` matches any known TDB command keywords (e.g., `ELEMENT`, `PHASE`). It allows for abbreviations and returns an integer code for the recognized keyword, setting `nextc` to the position after the keyword.
    *   **Parameters/Arguments:** `text` (Character, input line), `nextc` (Integer, position after keyword, OUT).

## Unformatted (Binary) Workspace Save/Read

*   **`gtpsave(filename, str)`**
    *   **Description:** A dispatcher routine that calls more specialized saving subroutines based on the format specified in `str`. Supported formats mentioned include Unformatted (`U`), Direct (`D`, not implemented), TDB (`T`, placeholder), LaTeX (`L`, not implemented), XTDB (`X`), and Macro (`M`, placeholder).
    *   **Parameters/Arguments:** `filename` (Character, output file path), `str` (Character, format specifier).

*   **`gtpsaveu(filename, specification)`**
    *   **Description:** Saves the entire current state of the GTP system to an unformatted binary file. This includes all elements, species, phases, model parameters, TP functions, defined equilibria with their conditions, state variable functions, and bibliographic references. It uses a workspace manager (`winit`, `wtake`, `storc`, `storr`, `storrn`) to pack data into a large integer array (`iws`) which is then written to the file. Includes versioning for data structures to maintain compatibility.
    *   **Parameters/Arguments:** `filename` (Character, output file path), `specification` (Character, further specification, unused).

*   **`savephases(phroot, iws)`**: Helper for `gtpsaveu` to serialize phase data.
*   **`saveequil(lok1, iws)`**: Helper for `gtpsaveu` to serialize equilibrium data.
*   **`svfunsave(loksvf, iws, ceq)`**: Helper for `gtpsaveu` to serialize state variable functions.
*   **`bibliosave(bibhead, iws)`**: Helper for `gtpsaveu` to serialize bibliographic references.
*   **`saveash(lok, iws)`**: Helper for `gtpsaveu` to serialize assessment data.

*   **`gtpread(filename, str)`**
    *   **Description:** Reads a GTP workspace from an unformatted binary file previously created by `gtpsaveu`. It reads the integer workspace array from the file and then calls various helper routines (`readphases`, `readequil`, etc.) to deserialize the data and reconstruct the GTP state. It performs version checks to ensure compatibility.
    *   **Parameters/Arguments:** `filename` (Character, input file path), `str` (Character, further specification, unused).

*   **`readphases(kkk, iws)`**: Helper for `gtpread` to deserialize phase data.
*   **`readendmem(lokem, iws, nsl, emrec)`**: Helper for `readphases` to deserialize endmember data.
*   **`readproprec(lokpty, iws, firstproprec)`**: Helper to deserialize property records.
*   **`readintrec(lokint, iws, level, intrec)`**: Helper to deserialize interaction records.
*   **`readequil(lokeq, iws, elope)`**: Helper for `gtpread` to deserialize equilibrium records.
*   **`svfunread(loksvf, iws)`**: Helper for `gtpread` to deserialize state variable functions.
*   **`biblioread(bibhead, iws)`**: Helper for `gtpread` to deserialize bibliographic references.
*   **`readash(lok, iws)`**: Helper for `gtpread` to deserialize assessment data.

## Placeholder/Unimplemented Save Routines

*   **`gtpsavelatex(filename, specification)`**: Placeholder for saving data in LaTeX format (not implemented).
*   **`gtpsavedir(filename, specification)`**: Placeholder for saving data in a direct access file format (not implemented).
*   **`gtpsavetm(filename, str)`**: Placeholder for saving data in a macro format (TDB or MACRO, not implemented here; TDB output is handled elsewhere).
*   **`gtpsavetdb(filename, specification)`**: Placeholder for saving data in TDB format (notes "Save TDB using gtpsavetdb not implemented"; actual TDB writing is likely via `list_TDB_format` in `gtp3C.F90`).

# Important Variables/Constants

*   **`savefile` (Character, from `gtp3.F90`):** A version string (e.g., `"OCSAVEF6"`) embedded in unformatted save files. This is checked by `gtpread` to ensure compatibility between the file and the current OpenCalphad version.
*   **`gtp_*_version` (Integer Parameters):** Version numbers associated with various GTP data structures (e.g., `gtp_element_version`, `gtp_phase_version`). These are stored in the unformatted save files and checked during reading to detect structural incompatibilities.
*   **TDB Command Keywords:** Implicitly, the `readtdb` routine and its helpers recognize standard TDB keywords like `ELEMENT`, `SPECIES`, `PHASE`, `CONSTITUENT`, `FUNCTION`, `PARAMETER`, `TYPE_DEFINITION`, etc.
*   **File Unit Numbers:** Standard Fortran integer unit numbers are used for file I/O (e.g., `21` for reading TDBs in `readtdb` and `checkdb2`, `31` for writing output in `list_many_formats` which can call `gtpsavetdb`).

# Usage Examples

*   **TDB Reading:** The `readtdb` subroutine is the primary interface for importing thermodynamic data from external TDB files into the OpenCalphad system. This is typically invoked via a user command like `READ_DATABASE my_database.TDB (ELEMENTS=FE,CR,NI)`.
*   **Workspace Save/Read:** `gtpsaveu` and `gtpread` are used for commands that allow saving the entire current session state (all loaded data, defined equilibria, etc.) to a binary file and later restoring it. This is much faster than re-reading TDB files. For example, `SAVE_WORKSPACE session_state.OCU` and `READ_WORKSPACE session_state.OCU`.

# Dependencies and Interactions

*   **GTP Core Data Structures (from `gtp3_dd1.F90`, `gtp3_dd2.F90`):**
    *   `readtdb` populates these structures by calling data entry routines (from `gtp3B.F90`).
    *   `gtpsaveu` reads from these structures to serialize their content.
    *   `gtpread` writes into these structures to deserialize the data.
*   **GTP Data Entry Routines (from `gtp3B.F90`):** `readtdb` makes extensive use of subroutines like `store_element`, `enter_species`, `enter_phase`, `enter_parameter`, etc., to build the in-memory thermodynamic database from the parsed TDB content.
*   **GTP Listing Routines:** `gtpsavetdb` (though a placeholder here) would conceptually use listing routines from `gtp3C.F90` to format data for TDB output.
*   **File I/O:** Directly performs Fortran file operations (OPEN, READ, WRITE, CLOSE, REWIND) for both text-based TDB files and unformatted binary workspace files.
*   **String Parsing Utilities (from `metlib`):** `readtdb` and its helpers likely use string manipulation and parsing functions (e.g., `getext`, `getrel`, `capson`) to decode TDB commands and their arguments.
*   **Workspace Manager (`winit`, `wtake`, `storc`, `storr`, `loadc`, `loadr` from `metlib` or `ocnum`):** The `gtpsaveu` and `gtpread` routines utilize these functions to manage the packing and unpacking of diverse data types into/from a large integer array (`iws`) for binary file storage.

This file is essential for data persistence and interoperability with the standard TDB file format.
