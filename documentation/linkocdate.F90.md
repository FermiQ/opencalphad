# Overview

The Fortran file `src/linkocdate.F90` contains a standalone program named `linkocdate`. Its primary purpose is to act as a build utility that automatically embeds the date of compilation (linking date) into the source code of the main OpenCalphad program file, `src/pmain1.F90`. This allows the compiled OpenCalphad executable to display its build date when run, which can be useful for version tracking and debugging.

The `linkocdate` program achieves this by:
1.  Fetching the current system date and time.
2.  Formatting this date into a 'YYYY-MM-DD' string.
3.  Reading a template/backup version of the main program (`src/pmain1-save.F90`).
4.  Searching for a specific placeholder string (`linkdate=`) within that file.
5.  Replacing the content after this placeholder with the formatted current date.
6.  Writing the modified content out to `src/pmain1.F90`, which is then used in the actual compilation of the OpenCalphad executable.

# Key Components

*   **`program linkocdate`**
    *   **Description:** The main program entry point. It orchestrates the date fetching, file reading, string manipulation, and file writing operations.
    *   **Logic:**
        1.  Calls the intrinsic subroutine `date_and_time(date)` to get the current date.
        2.  Formats the date string.
        3.  Opens `src/pmain1-save.F90` for reading (unit 21).
        4.  Opens `src/pmain1.F90` for writing (unit 22).
        5.  Reads lines from the input file one by one.
        6.  If a line contains the substring `linkdate=`, it modifies that line to include the current formatted date.
        7.  Writes the (potentially modified) line to the output file.
        8.  Closes both files.

# Important Variables/Constants

*   **`date` (character, length 20):** Stores the raw date and time string retrieved from the `date_and_time` intrinsic subroutine. Example: `20231027103045.000`
*   **`mdate` (character, length 12):** Stores the formatted date string intended for embedding. Example: `'2023-10-27'`
*   **`line` (character, length 60):** A buffer used to hold each line read from the input file (`src/pmain1-save.F90`).
*   **File Unit `21`:** Used for reading from `src/pmain1-save.F90`.
*   **File Unit `22`:** Used for writing to `src/pmain1.F90`.
*   **`'linkdate='` (Literal string):** The specific placeholder string that `linkocdate` searches for in `src/pmain1-save.F90` to identify the line where the date should be inserted.

# Usage Examples

The `linkocdate` program is not a library subroutine called by other parts of OpenCalphad at runtime. Instead, it is a utility that is executed as part of the OpenCalphad build process, typically orchestrated by a Makefile or a build script.

**Conceptual Build Process Steps:**

1.  **Compile `linkocdate.F90`:**
    ```bash
    gfortran -o linkocdate_executable src/linkocdate.F90
    ```
2.  **Ensure `src/pmain1-save.F90` exists:** This file serves as the input template. It might be a copy of `src/pmain1.F90` from a previous build or a clean template file.
3.  **Run the compiled `linkocdate_executable`:**
    ```bash
    ./linkocdate_executable
    ```
    This step reads `src/pmain1-save.F90`, inserts the current date, and writes the output to `src/pmain1.F90`.
4.  **Compile the main OpenCalphad program:** The build process then compiles `src/pmain1.F90` (which now contains the embedded date) along with the rest of the OpenCalphad source code.

The `src/pmain1.F90` file itself would typically have a line similar to:
```fortran
character(len=10) :: linkdate = 'YYYY-MM-DD' ! Initial value, will be replaced
! ... later in pmain1.F90 ...
write(*,*) 'OpenCalphad linked on: ', linkdate
```
The `linkocdate` program modifies the `'YYYY-MM-DD'` part.

# Dependencies and Interactions

*   **Input File (`src/pmain1-save.F90`):** `linkocdate` requires this file to exist and to serve as the source template for `src/pmain1.F90`. The content of this file, especially the line containing `linkdate=`, is critical.
*   **Output File (`src/pmain1.F90`):** This file is created or overwritten by `linkocdate`. It becomes the version of the main program source code that includes the embedded linking date and is subsequently compiled.
*   **Build System (e.g., Makefile):** The execution of `linkocdate` is managed by the project's build system. The build system ensures it runs before the main program `pmain1.F90` is compiled.
*   **`pmain1.F90` (Main OpenCalphad program):** While `linkocdate` modifies `pmain1.F90`, the main program `pmain1.F90` consumes the date by having a character variable (likely also named `linkdate`) that gets initialized with the string value inserted by `linkocdate`. This allows `pmain1.F90` to display the link date at runtime.
