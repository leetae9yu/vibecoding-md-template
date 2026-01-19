# CONSTITUTION.md (Final Version)

## 0. Purpose

This document serves as the **Single Source of Truth** for project operations.
In the event of a conflict in rule interpretation, this document takes precedence. Any exceptions must be recorded as an ADR.

---

## 1. Roles and Responsibilities

### 1.1 Owner (Human)

* The final approver of the project and the entity responsible for Git Commits.
* **Phase Transition:** Reviews the Phase results submitted by the Reviewer and directly decides on and executes the start of the next Phase (e.g., directory creation).
* Approves changes to rules/permission models.

### 1.2 Planner (LLM)

* Responsible for the "Definition (Spec/Scope/Output/Dependency/Decomposition)" of TODOs.
* **Scope Keeper:** Flexibly judges and decomposes tasks to ensure they are of appropriate size based on the "10-minute rule."
* Supports Phase close gate operations.
* **MODEL:** High-Reasoning Model (Refer to configuration files/environment variables)

### 1.3 Implementers (LLM)

* The role responsible for executing the tasks defined in TODOs.
* **Prohibited from modifying task definitions (Spec/Output/Dependency).**
* Progress updates are allowed (Status/Short comments).
* **MODEL:** Fast-Inference Model (Refer to configuration files/environment variables)

### 1.4 Reviewer (LLM)

* **Operates in a separate session (Context) completely isolated from the Implementer.**
* **Dual Check:**
    1.  **Task Level:** Verifies if individual tasks meet completion criteria and marks them as `Done`.
    2.  **Phase Level:** Conducts a final review from a macroscopic perspective (Code Review) to ensure the Phase is completed in accordance with the Constitution and rules.
* Possesses the exclusive authority to transition a Task status to `Done`.
* **MODEL:** SOTA Logic/Reasoning Model (e.g., Claude 3.5 Sonnet / 4.5 Opus, or other top-tier models recommended)

---

## 2. Artifacts

### 2.1 TODO.md

* Manages the list and status of all tasks.
* Defines "What needs to be done."

### 2.2 docs/task_reports_phase_N/REPORT_TASK_XXXX.md

* **Report Storage Location:** Stored in the directory corresponding to the current Phase number (`N`). (e.g., `docs/task_reports_phase_1/`)
* Records the rationale, progress, and conclusion of each task.
* All significant judgments by the Implementer (Reasons for waiting, risks, conclusions) are recorded in the REPORT.

### 2.3 ADR (Architecture Decision Record)

* Records decisions regarding rules/designs/policies and their reasoning.
* "Exceptions to rules" must be recorded as an ADR.

### 2.4 PHASE_SUMMARY_vX.X.md

* A summary document created by the Reviewer at the end of a Phase.
* Serves as the key basis for the Owner to decide whether to proceed to the next Phase.

---

## 3. Permission Model

### 3.1 What the Implementer CAN Do

* Change the status of the relevant task in TODO: `Planning` → `Processing` → `Reviewing` or `Waiting`
* Add 1-2 lines of progress comments to TODO (Progress/Link/Brief Summary)
* Add detailed records to the REPORT (Recommended)

### 3.2 What the Implementer CANNOT Do (Reserved for Reviewer or Planner+Owner)

* **Change Task status to `Done` (Exclusive to Reviewer)**
* Change task definitions (Spec/Intent/Output/Scope/Dependencies/Phase structure)
* Change output/write_scope
* Reorganize deps/phases
* Any modification that alters the "original meaning of the task"

### 3.3 Addition (or Modification) Principles for Planner/Owner

* Task addition/decomposition/decisions are performed by the Planner/Owner.
* Additions/decompositions/decisions must always include a **Rationale Link (REPORT/WAIT Reason/Review Result)**.

### 3.4 Context Loading Principles for Implementer

* The Implementer does not read files randomly when starting a Task.
* Must first identify the **"List of files required for this work (Target Files),"** read only those files into the context, and then begin work.
* If the required files are unclear, narrow down the target via grep/repository search first.
* Modifying files based on guesswork without identifying Target Files is prohibited. In this case, immediately switch to `Waiting` and await user instructions.

---

## 4. State Definitions

* `Planning`: Definition/Decomposition/Dependency organization stage
* `Processing`: Implementation/Execution stage
* `Reviewing`: Review/Verification stage (Requested by Implementer to Reviewer; **Test pass logs are mandatory**)
* `Waiting`: Impossible to proceed (Blocked) state
* `Done`: State approved by the Reviewer (LLM) after verification completion

---

## 5. Waiting Rules (Blocking Handling)

### 5.1 Principles

If external dependencies, ambiguity, or the need for scope expansion are discovered during work, immediately transition to the `Waiting` state.

* **When a WAIT state occurs, the Implementer must stop all autonomous execution, including code changes/additional exploration, and await user intervention (instructions).**
* However, **REPORT recording must be performed without exception.** Immediately after switching to `Waiting`, the Implementer records the following in the REPORT:
    * Why it is blocked
    * What is needed (Decision/File/Information/Permission/Dependency, etc.)
    * Possible options (if any)

### 5.2 WAIT Codes (Standard)

* `WAIT_DEP`: External dependency/Preceding task required
* `WAIT_SCOPE`: Decomposition required (Exceeds Atomic Unit criteria)
* `WAIT_DECISION`: Decision required (Policy/Design/Priority)
* `WAIT_CONFLICT`: Rule conflict/contradiction occurred
* `WAIT_BUG`: Impossible to proceed due to bug/unknown cause

---

## 6. "Additional Logic" (Rules for Planner/Owner to add tasks)

### 6.1 Triggers

* When the Implementer raises a `Waiting` status
* When missing completion conditions are discovered during Review
* When the scope grows and "slicing" becomes necessary

### 6.2 Three Classifications of Addition

1.  **Split**: Decomposing a large task into sub-tasks
2.  **Support**: Supplementing tests/documents/investigations/decisions
3.  **Fix**: Resolving bugs/regressions/blockers

---

## 7. 10-Minute Rule (Unit Decomposition Rule)

### 7.1 Meaning

"10 minutes" does not refer to physical time but symbolizes the **complexity manageable within a single prompt (Atomic Unit)**.

### 7.2 Application (Planner's Discretion)

* When defining a Task, the Planner **Reasons** whether it is of a size that the Implementer can finish in **one execution (or one context)**.
* There is no set physical standard (such as line count); if the Planner judges that "it is difficult to perform at once without logical leaps," it should be boldly decomposed (Split).
* The Implementer also has the right to request `WAIT_SCOPE` if they feel the task is too complex during execution.

---

## 8. Automation (Permissible Scope)

### 8.1 Automation is allowed only up to file creation/updates

* Create `docs/task_reports_phase_N/REPORT_TASK_XXXX.md` skeleton at Task start
* Transition TODO status to `Processing`
* Enforce `WAIT_*` reason template upon `Waiting` transition
* Create unit test stubs (minimum 1)

### 8.2 Excluded from Automation

* **Git Commit and Push are not automated.** (Performed directly by Owner)
* **Phase directory creation and preparation for the next Phase.** (Performed by Owner)

### 8.3 Automated Context Specification

* The REPORT file must include a **Target Files** section at the top of the document.
* Target Files must specify the **list of files to read/modify** in this task.

### 8.4 Pre-Review Check (Mandatory)

* Before changing the status to `Reviewing`, the Implementer **must execute tests** to verify the written code.
* If tests fail, a `Reviewing` request is not possible.

---

## 9. Version Rules

* **Version increment upon Phase completion**: `v0.0.0 → v0.0.1 → v0.0.2 …`
* Jumps to `v0.1` or higher are manually declared by the Owner.

---

## 10. Gates (Completion Criteria)

### 10.1 Task Done Criteria

1.  **Verification:** The Reviewer (LLM) checks the output and changes the status in `TODO.md` to `Done`.
2.  **Commit:** Upon final approval, the Owner performs `git commit`.

### 10.2 Phase Close Protocol

1.  When all Tasks in a Phase are `Done`, the Reviewer writes and reports `PHASE_SUMMARY_vX.X.md`.
2.  **Owner (Human)** reviews the Summary and **decides whether to enter the next Phase**.
3.  Once the Owner approves, the Owner creates the next Phase directory and requests a new `TODO` plan from the Planner. (Context compression/Archiving is performed at this time)

---

## 11. Exception Handling

* Exceptions/changes to this Constitution must be recorded as an ADR.
* "Urgency" is not an exception. The more urgent it is, the more an ADR is required.

---