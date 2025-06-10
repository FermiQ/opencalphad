# Overview

The Python script `pyOC.py` provides a high-level, object-oriented interface for interacting with the OpenCalphad thermodynamic library. It acts as a more Pythonic wrapper around the lower-level `f2py`-generated module (presumably named `rawpyOC`, which itself wraps Fortran code from `pyOC.f90` and `liboctq.F90`). The main goal of `pyOC.py` is to offer a user-friendly and convenient API for Python developers to perform thermodynamic calculations, manage system conditions, and retrieve results from OpenCalphad.

It defines several classes, including `OpenCalphad` which is the primary entry point, `PhasesAtEquilibrium` for structuring results, and `IntEnum` based classes for statuses.

# Key Components

## Enumerations

*   **`PhaseStatus(IntEnum)`**
    *   **Description:** Defines symbolic names for phase statuses used by OpenCalphad.
    *   **Members:** `Suspended (-3)`, `Dormant (-2)`, `Entered (0)`, `Fixed (2)`.
    *   **Purpose:** Improves code readability when setting or interpreting phase statuses by using names instead of raw integers.

*   **`GridMinimizerStatus(IntEnum)`**
    *   **Description:** Defines symbolic names for the grid minimizer's operational status.
    *   **Members:** `On (0)`, `Off (-1)`.
    *   **Purpose:** Provides a clear way to control the grid minimizer during equilibrium calculations.

## Data Classes

*   **`PhasesAtEquilibrium(object)`**
    *   **Description:** A class designed to hold and structure detailed information about all phases present at equilibrium.
    *   **Constructor:** `__init__(self, phaseMolarAmounts, phaseElementComposition, phaseSites, phaseConstituentComposition)`
        *   Initializes the object with dictionaries and lists containing data about phase amounts, elemental compositions, site fractions, and constituent compositions.
    *   **Methods:**
        *   `getPhaseMolarAmounts()`: Returns a dictionary of phase names to their molar amounts.
        *   `getPhaseElementComposition()`: Returns a nested dictionary of phase names to element names to their mole fractions.
        *   `getPhaseSites()`: Returns a dictionary of phase names to their site fraction arrays.
        *   `getPhaseConstituentComposition()`: Returns a nested dictionary detailing constituent compositions within phases (and sublattices if present).
        *   `__str__()`: Provides a JSON-formatted string representation of all contained data for easy inspection.

## Main Interface Class

*   **`OpenCalphad(object)`**
    *   **Description:** The primary class that Python users interact with. It encapsulates the connection to the underlying `f2py`-generated OpenCalphad functions and manages the state of equilibrium calculations.
    *   **Constructor:** `__init__(self)`
        *   Sets up logging for the OpenCalphad interface.
        *   Initializes internal dictionaries like `__equilibriumNamesInOC` to manage multiple equilibrium records and `__constituentsDescription` to cache information about constituents.
    *   **Methods:**
        *   `setVerbosity(self, isVerbose)`: Controls the verbosity of both the Python logger and the underlying Fortran library's output (via `oc.pytqquiet`).
        *   `raw(self)`: Provides access to the raw `f2py`-generated module (`rawpyOC.rawopencalphad`, aliased as `oc`). Useful for advanced users or debugging.
        *   `eq(self)`: Returns the current low-level equilibrium object/pointer being used by the `rawpyOC` module.
        *   `readtdb(self, tdbFilePath, elements=None)`: Initializes a new default equilibrium calculation, reads a Thermo-Calc Database (TDB) file. It can read all elements or a specified list. After reading, it retrieves and stores component names.
        *   `getComponentNames(self)`: Returns a list of component (element) names in the system.
        *   `getConstituentsDescription(self)`: Returns a dictionary containing descriptions (mass, charge, elemental stoichiometry) of constituents encountered during calculations.
        *   `setPhasesStatus(self, phaseNames, phaseStatus, phaseAmount=0.0)`: Sets the thermodynamic status (e.g., `PhaseStatus.Suspended`, `PhaseStatus.Fixed`) of one or more phases.
        *   `setTemperature(self, temperature)`: Sets the system temperature as a condition for equilibrium calculation.
        *   `setPressure(self, pressure)`: Sets the system pressure as a condition.
        *   `setElementMolarAmounts(self, elementMolarAmounts)`: Sets the molar amounts of elements in the system. `elementMolarAmounts` is a dictionary mapping element symbols to their amounts.
        *   `changeEquilibriumRecord(self, eqName=None, copiedEqName=None)`: Allows managing multiple equilibrium states. It can create a new named equilibrium record by copying an existing one or switch the active equilibrium record.
        *   `calculateEquilibrium(self, gridMinimizerStatus=GridMinimizerStatus.On)`: Triggers the core equilibrium calculation in the Fortran backend.
        *   `getErrorCode(self)`: Returns the error code from the last operation in the Fortran backend.
        *   `resetErrorCode(self)`: Resets the error code in the Fortran backend.
        *   `getScalarResult(self, symbol)`: Retrieves a single scalar result (e.g., 'G' for Gibbs energy, 'T' for temperature) from the calculated equilibrium.
        *   `getGibbsEnergy(self)`: A convenience method that calls `getScalarResult('G')`.
        *   `getComponentAssociatedResult(self, symbol)`: Retrieves results that are associated with each component (e.g., 'MU' for chemical potentials). Returns a dictionary mapping component names to values.
        *   `getChemicalPotentials(self)`: A convenience method that calls `getComponentAssociatedResult('MU')`.
        *   `getPhasesAtEquilibrium(self)`: A comprehensive method that retrieves data for all stable phases at equilibrium. This includes their molar amounts, elemental compositions, site fractions, and constituent compositions. It populates and returns a `PhasesAtEquilibrium` object. It also updates the internal `__constituentsDescription` cache.

# Important Variables/Constants

*   **`OpenCalphad._defaultEquilibriumName` (Class attribute):** `"default equilibrium"`. Used as the name for the initial equilibrium record created by `readtdb`.
*   **`OpenCalphad._maxNbPhases`, `_maxNbElements`, `_maxNbSublattices`, `_maxNbConstituents` (Class attributes):** Integers (400, 100, 10, 100 respectively). These define the maximum dimensions for arrays pre-allocated with `numpy.empty` before being passed to the underlying `rawpyOC` functions. This is a common pattern when interfacing Python with Fortran code that expects fixed-size arrays or needs buffer information.
*   **`opencalphad` (Module-level instance):** `opencalphad = OpenCalphad()`. A global instance of the `OpenCalphad` class is created, making it immediately available for import and use by other Python scripts.

# Usage Examples

```python
# Conceptual Python usage example
from pyOC import opencalphad, PhaseStatus, GridMinimizerStatus # Assuming pyOC.py is in python path

# Initialize and read a database
elements_to_read = ["FE", "C", "MN"]
try:
    opencalphad.setVerbosity(True) # Enable detailed logging
    opencalphad.readtdb("path/to/your/database.tdb", elements=elements_to_read)

    print("Components:", opencalphad.getComponentNames())

    # Set conditions
    opencalphad.setTemperature(1873.15) # Temperature in Kelvin
    opencalphad.setPressure(101325.0)   # Pressure in Pascals
    opencalphad.setElementMolarAmounts({"FE": 0.98, "C": 0.02, "MN": 0.01}) # Example molar amounts

    # Set phase status (e.g., suspend a phase)
    # opencalphad.setPhasesStatus(["GRAPHITE"], PhaseStatus.Suspended)

    # Calculate equilibrium
    opencalphad.calculateEquilibrium(gridMinimizerStatus=GridMinimizerStatus.On)

    error_code = opencalphad.getErrorCode()
    if error_code != 0:
        print(f"Calculation Error Code: {error_code}")
    else:
        print("Equilibrium calculated successfully.")
        gibbs_energy = opencalphad.getGibbsEnergy()
        print(f"System Gibbs Energy: {gibbs_energy} J/mol")

        chemical_potentials = opencalphad.getChemicalPotentials()
        print("Chemical Potentials:", chemical_potentials)

        phases_info = opencalphad.getPhasesAtEquilibrium()
        print("Phases at Equilibrium:")
        print(phases_info) # Uses the __str__ method of PhasesAtEquilibrium

        # Example of creating a new equilibrium record and modifying it
        # opencalphad.changeEquilibriumRecord(eqName="experiment_2", copiedEqName="default equilibrium")
        # opencalphad.setTemperature(1973.15)
        # opencalphad.calculateEquilibrium()
        # print(f"Gibbs Energy (Experiment 2): {opencalphad.getGibbsEnergy()}")

except Exception as e:
    print(f"An error occurred: {e}")
    print(f"Underlying OC error code: {opencalphad.getErrorCode()}")

```

# Dependencies and Interactions

*   **`rawpyOC` (Python Module):** This is the core dependency. `pyOC.py` assumes the existence of a module named `rawpyOC`, which is the `f2py`-generated Python extension module created from `OCisoCbinding/pyOC/pyOC.f90` and its Fortran dependencies. The `rawpyOC.rawopencalphad` submodule is specifically imported and aliased as `oc`.
*   **`numpy` (Python Package):** Used extensively for creating and managing numerical arrays (e.g., `np.empty`, `np.array`, `np.where`) that are passed to and from the `rawpyOC` module.
*   **`json` (Python Standard Library):** Used for formatting dictionary outputs into JSON strings, primarily for logging and the `__str__` method of `PhasesAtEquilibrium`, enhancing readability of complex data structures.
*   **`logging` (Python Standard Library):** Used to provide configurable logging messages, allowing users to control the verbosity of the `OpenCalphad` class operations.
*   **`enum.IntEnum` (Python Standard Library):** Used as the base class for `PhaseStatus` and `GridMinimizerStatus` to create type-safe enumerations.
*   **`sys` (Python Standard Library):** Used by the logging setup to direct log messages to `sys.stderr`.
*   **Interaction Flow:** Python User -> `pyOC.py` (`OpenCalphad` class) -> `rawpyOC` (f2py module) -> `pyOC.f90` (Fortran wrappers) -> `liboctq.F90` (Fortran core TQ routines) -> OpenCalphad engine.
