# verified_md

verified_md is a documentation convention for software projects that are
verified by mechanical checks. A project that follows the convention has a
file named `VERIFIED.md` at the root of its repository. The file lists the
properties of the software that the checks support.

A reader with a question about the behaviour of the software reads
`VERIFIED.md` first. The reader can treat each statement in the file as
true, within the bound that the statement gives. The reader is a human or
an LLM.

## Purpose

A verified project contains much more text than a reader needs to answer
most questions about it: source code, models, test suites and their
documentation. The conclusions of the verification are a small part of
that text.

`VERIFIED.md` holds those conclusions in one file. The file serves two
purposes:

- **Compression.** A reader, or the context of an LLM, holds one file and
  not the whole repository.
- **Trust.** Each statement in the file names the check that supports it.
  A statement in other documentation is a description. A statement in
  `VERIFIED.md` is a description that a check has tested.

## Structure of a project

| File | Content | Size |
|---|---|---|
| `README.md` | What the project is, an example, and a link to `VERIFIED.md`. | Short |
| `VERIFIED.md` | The verified properties, their bounds, the assumptions, and the properties that are not verified. | One file. The whole file fits in the context of an LLM. |
| Detailed documentation | How each verification method works, its results and its commands. | Any |

The entry in `README.md` states three things:

1. `VERIFIED.md` lists the properties that a mechanical check supports.
2. A reader with a question about the behaviour of the software reads
   `VERIFIED.md` first.
3. A statement in `VERIFIED.md` is true within the bound that its entry
   states.

## Requirements for a project

A project can follow the convention when all of these hold:

- The project has at least one check that is stronger than example-based
  unit tests. Examples are model checking, a machine-checked proof,
  deterministic simulation with invariants, property-based testing, and an
  independent checker such as a linearizability checker.
- Each check runs from a documented command.
- Each check can fail. The project shows this with mutations or planted
  bugs that the check rejects.
- The maintainers accept the maintenance rules below.

## Sections of VERIFIED.md

`VERIFIED.md` has these sections, in this order.

| Section | Content |
|---|---|
| Header | The name and the version of the project, and links to the detailed documentation. The header does not name a commit. See "Versions". |
| How to read this file | The rules for a reader. See "Rules for a reader". |
| Methods | One row for each verification method: the check, what a pass shows, and what a pass does not show. |
| Assumptions | The conditions under which every entry holds. Examples are durable storage, a fault model and clock bounds. |
| Dependencies | The software that the project depends on, the entries of each dependency that the project relies on, and how each check treats the dependency. See "Dependencies". |
| Verified properties | The entries, in tables grouped by subject. |
| Evidence that the checks can fail | The mutations and planted bugs that each method rejects. |
| Not verified | The properties that a reader is likely to assume and that no check supports. |
| Limitations | The known weaknesses of the methods. |
| Reproduction | The command, the duration and the date of the last recorded run for each method. |
| Maintenance | The rules that keep the file true. |

## Format of an entry

An entry is one row of a table with four columns.

| Column | Content |
|---|---|
| Id | A short identifier, such as `C1`. The identifier does not change when the file is edited, so other documents and conversations can cite it. |
| Property | One statement about the behaviour of the software, in plain words. The statement names the invariant or the test that states it formally, if one exists. |
| Method | The methods that support the property. Each method is defined in "Methods". |
| Bound | The scope in which the check ran: the constants of a model, the workloads and the number of runs, or the name of a test. |

Example, for a consensus library. The methods `TLC` and `TRACE` are
defined in the "Methods" section of that file.

| Id | Property | Method | Bound |
|---|---|---|---|
| `C1` | Agreement. At most one value is decided for each index. | `TLC` | 3 to 5 acceptors, 2 to 3 epochs, 1 to 2 indexes |
| `K1` | A run of the code is a behaviour of the model `Consensus.tla`. | `TRACE` | 250 simulated runs |

## Rules for an entry

- **A check supports the entry.** A statement that rests only on an
  argument, on a design document or on code review is not an entry.
- **The check can fail.** A check with no mutation or planted bug that it
  rejects supports no entry.
- **The entry states its bound.** "Agreement holds" is not an entry.
  "Agreement holds in the model for 3 acceptors, 2 epochs and 2 indexes"
  is an entry.
- **The entry states what was checked: a model or the code.** A property
  of a model is not a property of the code. A project that connects the
  two, for example with trace validation or refinement, states that
  connection as an entry of its own.
- **The entry distinguishes exhaustive checks from tests.** A model
  checker visits every state within its bound. A simulation checks the
  runs that its seeds produce.
- **The entry is one statement.** A reader can cite the entry and can
  decide whether it answers a question.
- **The entry uses the terminology of the project.**

## Dependencies

A project can depend on another project that has a `VERIFIED.md`. The
"Dependencies" section of the dependent project states which entries of
the dependency the project relies on. A reader can then follow a property
of the dependent project to the checks of the dependency.

The section is a table with one row for each dependency.

| Column | Content |
|---|---|
| Dependency | The name of the dependency and a link to its `VERIFIED.md`. |
| Version | The version of the dependency that the project uses: a tag, a reference to the lock file of the build, or the name of a branch if the build fixes no version. |
| Entries relied on | The identifiers of the entries of the dependency that the project relies on. |
| Used for | What the project uses the dependency for, and the configuration in which the project uses it. |
| Treatment | How each method of the project treats the dependency. See the next table. |

A method treats a dependency in one of three ways.

| Treatment | Meaning | Strength of the result |
|---|---|---|
| Assumed | The check contains an abstraction of the dependency. For example, a model represents a replicated log as one atomic variable. | The result holds only if the entries relied on hold. |
| Included | The check runs the code of the dependency. For example, a simulation starts a cluster of the dependency. | The result covers the dependency, in the runs of the check. |
| Replaced | The check runs a substitute for the dependency, such as a stub. | The result states nothing about the project together with the dependency. |

Example, for a project that stores its state in a cluster of a consensus
library named `logd`:

| Dependency | Version | Entries relied on | Used for | Treatment |
|---|---|---|---|---|
| [logd](https://example.org/logd/VERIFIED.md) | 2.4.0 | `C1`, `K2`, `K5`, `L1` | A linearizable store of the coordinated state. 3 servers, majority quorums, no read leases. | `TLC`: assumed. `SIM`: included. `UNIT`: replaced in 12 tests, included in the others. |

Rules for the section:

- **The row cites entries.** A row that names a dependency and no entry
  states nothing that a reader can check.
- **The project uses the dependency within the bound of each entry.** An
  entry of a consensus library about majority quorums does not support a
  project that uses the library with grid quorums. The "Used for" column gives the configuration, so
  that a reader can compare it with the bound.
- **An entry of the project is not stronger than the entries it relies
  on.** An entry whose method treats the dependency as assumed holds only
  within the bounds of the entries relied on.
- **The project inherits the assumptions of the dependency.** The
  "Assumptions" section of the project repeats each inherited assumption
  or refers to it by its identifier in the dependency.
- **The row names a version.** The entries relied on are the entries of
  the dependency at that version. A project that moves to a later version
  of the dependency compares the entries again. A row that names a branch
  states that the build fixes no version.
- **A removed or narrowed entry affects the dependent project.** If the
  dependency removes an entry or narrows its bound, the dependent project
  removes or narrows each entry that relies on it.
- **A dependency without a `VERIFIED.md` is listed.** The section has a
  second table for these dependencies, with the behaviour that the project
  assumes of each one. Examples are a runtime system, an operating system
  and a test tool. An assumption in "Assumptions" refers to the table. A
  dependency that is not listed is not verified.

An entry under "Verified properties" that relies on a dependency names the
dependency in its "Bound" column, for example "Assumes logd `C1`".

## Versions

`VERIFIED.md` does not name the commit that it describes. The file is in
version control with the code. The maintenance rules require that a change
to a check and the change to `VERIFIED.md` are in the same commit. The file
at a commit therefore describes the software at that commit.

The file records a date, and a commit if one is known, in two places:

- **The last recorded run of a check, in "Reproduction".** This applies to
  a check that continuous integration does not run on each change, such as
  a model checking run of several hours. The date tells a reader how old
  the evidence is. A date of a past run does not become false when the
  code changes.
- **The version of a dependency, in "Dependencies".**

## Rules for a reader

1. A statement under "Verified properties" is true for the version of the
   software in the same commit as the file, within its bound and under the
   assumptions.
2. A statement is not verified beyond its bound. The statement can be
   true or false there. A reader who extends a statement says that the
   extension is not verified.
3. A property that is not in the file is not verified. The property can
   be true or false. The file does not state that the property is false.
   The absence of a property from "Not verified" does not make the
   property verified.
4. If the file and the code disagree, the file is out of date. The reader
   reports the disagreement and trusts the checks, which run on the code.
5. A reader who cites the file gives the identifier of the entry.
6. An entry that relies on a dependency is verified only within the bounds
   of the entries relied on, and under the assumptions of the dependency. A
   reader who needs the bound reads the `VERIFIED.md` of the dependency at
   the version that the row names.

An LLM agent that works in a repository with a `VERIFIED.md` reads the file
before it searches the code for an answer about the behaviour of the
software. The agent reads the code when the file does not answer the
question.

## Maintenance rules

1. A change that adds, removes or weakens a check updates `VERIFIED.md` in
   the same commit.
2. A change that makes an entry false removes the entry or narrows its
   bound in the same commit.
3. An entry is added only when a check that can fail supports it.
4. A run of a check that continuous integration does not run on each
   change updates the date of the last recorded run in "Reproduction".
5. A reviewer of a change to `VERIFIED.md` compares each changed entry
   with the output of its check.

## Limitations

- **The file is a claim by the maintainers.** No tool checks that an entry
  matches the check that it names. A reader trusts the file as far as the
  reader trusts the maintenance of the project.
- **The file can be out of date.** A change to the code that does not
  follow the maintenance rules leaves an entry that the code no longer
  satisfies.
- **A recorded run can be old.** A check that continuous integration does
  not run on each change supports its entries only as of the date of its
  last recorded run.
- **The file does not make a bounded check unbounded.** A reader who
  ignores the bounds concludes more than the checks show.
- **The file does not replace the detailed documentation.** It states what
  was verified and not how the software works.
- **No check supports the composition of two projects.** A project whose
  model treats a dependency as assumed states that the entries of the
  dependency provide what the model assumes. That statement is an
  argument. A check that treats the dependency as included tests the
  composition in the runs of the check.

## Examples

| Project | Description | Methods | Dependencies with a `VERIFIED.md` |
|---|---|---|---|
| [peace](../peace/VERIFIED.md) | An Erlang library for replicated state machines that implements Flexible Paxos | Model checking, an inductive invariant, trace validation, deterministic simulation, linearizability checking, a Jepsen test, unit tests | None |
| [redwall](../redwall/VERIFIED.md) | A transactional key-value store in Erlang that implements the transaction system of FoundationDB | Model checking, deterministic simulation, unit tests | peace |

The "Dependencies" section of redwall names 12 entries of peace. Its
models treat peace as assumed. Its simulation treats peace as replaced in
43 tests and as included in 9 tests.

The README of each project has a section "Verified properties" that links
to the file.
