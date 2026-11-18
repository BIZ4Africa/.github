# BIZ4A coding standards

One copy, for every repository of the organisation. A project repository does not carry
its own copy of these rules: it points here.

Every rule below is either enforced by a script (the CI runs it) or stated with the
reason it cannot be automated yet. When an instruction from the delivery owner
contradicts a rule here, the instruction wins and this file is corrected.

## 1. Language

**Code is written in English.** Identifiers, docstrings, comments (SQL comments
included), migration docstrings, log messages, exception messages we raise, file names.

**Everything a user reads is translated, not written in one language.** A label, a help
text, a report title, a receipt footer goes through Odoo's translation mechanism
(`self.env._(...)`, `t-out="_('...')"`), with the English string as the source and the
French text in `i18n/fr.po`. That is what makes a document print in the **partner's**
language. Hard-coding French breaks that native behaviour exactly as much as hard-coding
English does.

Three things are deliberately **not** translated, and only these three:

| Exemption | Why |
| --- | --- |
| Regulatory vocabulary whose wording is prescribed by the authority (tax group names, DEF, NIM, ISF, labels imposed by the norm) | translating it would change a regulatory reference |
| A verbatim message produced by a third party (connector, platform) quoted in a docstring or a test | a quote is evidence: translated, it no longer matches what the user sees |
| Data whose value must match a record in the database (payment method names, partner names) | it is data, not prose |

**Scope.** The rule covers what the *client's* users read: screens, documents, receipts,
invoices, errors shown in the interface. The console output of internal provisioning
tooling is English, because its reader is the delivery team.

Enforced by `tools/check_code_language.py`, as a ratchet against a baseline file: a file
above its recorded count fails the build, a file below passes and can have its baseline
lowered, and a new file must be clean. The baseline only ever goes down.

## 2. Repository and branch scope

- A **product** repository implements the standard capability (a localisation, a fiscal
  interface). It is shared by every client: it names no client, no brand, no client
  instance, no integrator.
- A **client** repository carries that client's configuration: modules, data, programme
  documents.
- A defect found in a repository outside the current mission is **filed as an issue
  there**, with its measurement. It is not fixed from another repository. The only
  exception is a data leak or a security flaw, which is treated immediately and declared.
- Work happens on **the branch the owner names**, and only that one, until the owner
  orders a promotion. A merge into a downstream branch is a **decision**, never a
  consequence of a build. Drift between branches is **reported**, never enforced as a
  gate: a guard must never push an agent to touch a branch it was told to leave alone.
- **No module is created without an explicit request.** A module created on a
  description rather than on a measurement becomes dead weight: measure what the code
  does before splitting or adding.

## 3. Shared modules stay neutral

A shared base module names **no** company, no brand, no project. Each specific module
designates itself (a flag set on its own record from its own data file); the base reads
the flag. A rule that mentions a name is either generic (rewrite it) or belongs to the
module it names (move it).

## 4. Configuration, not infrastructure

A module repository contains modules and the documents that describe them. It contains
**no container command, no host path, no deployment script**. Deploying, restarting and
updating an environment is the platform's job.

Nothing runs inside an environment container: no `docker exec`, no `psql` in the database
container, no edit under the environment's mounted tree. The authorised path is the
platform API and Odoo RPC on the public URL. A checkout edited by hand is overwritten by
the next deployment.

## 5. Tests

- a test proves a behaviour, not the shape of a function;
- a test never fabricates what it needs from the environment: it uses the product's own
  code paths, and skips when the field it exercises is absent;
- a history of builds and incidents is not test documentation: it belongs to the tracker.

## 6. Issues and commits

- one issue = one subject, in the repository that owns the code, written in English, with:
  measured evidence, root cause with `file:line`, the fix, **what it does not prove**, and
  verifiable acceptance criteria;
- an issue is labelled `Processing` only while it is actively being worked, and the label
  comes off when the work stops;
- closing requires a **signed** comment: who, when, why, and the evidence. `not planned`
  is reserved for a decision taken by the owner, never for an arbitration of our own;
- commits are English, imperative, and say why rather than what.

## 7. Guard scripts

Each repository runs the subset that applies to it:

| Script | What it refuses |
| --- | --- |
| `check_code_language.py` | French in code, beyond the ratchet baseline |
| `check_shared_module_names.py` | a specific name inside a shared base |
| `check_client_neutrality.py` | a client identifier in a product repository (denylist from a CI secret, never committed) |
| `check_conflict_markers.py` | unresolved Git conflict markers |
| `check_manifests.py` | an unreadable manifest, or a version from another Odoo series |
| `check_enterprise_deps.py` | an Enterprise module in the Community closure |
| `check_no_test_accounts.py` | a test account in a production-bound data file |

A guard that fails on a pre-existing condition is corrected by fixing the code, never by
adding a tolerance — except a language ratchet, whose baseline is a measured debt that may
only shrink.
