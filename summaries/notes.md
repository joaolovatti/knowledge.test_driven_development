# Testing Fundamentals with pytest

*Replace hope with fast, repeatable checks: assertions that catch regressions before production.*

## In short

- Coding, clicking around and "hoping nothing broke" leads to fear of refactoring, technical debt and a fragile codebase.
- A **test** checks behavior against an expectation using an **assertion**; in Python, `assert`.
- Tests are a **regression** safety net: they catch changes that break what used to work.
- **pytest** is the modern Python standard: plain `assert`, little boilerplate, discovery by naming convention.
- A failing test is a precise, automated bug report: read the indicator, the message and the location.

## Why testing matters

The "default" workflow is intuitive but risky: have an idea, write a lot of code, run it manually, click around, then deploy and hope nothing broke. That reliance on hope is where the risk lives.

Familiar symptoms follow: **fear of refactoring** your own code, and "it worked on my machine".

Over time this creates **technical debt**. The codebase grows messier, changes get slower, velocity drops, and the team loses confidence in the system: a vicious cycle of increasing fragility.

Breaking the cycle means replacing hope with systematic testing. When tests come first, as in **test-driven development (TDD)** and its red, green, refactor cycle, they also guide design and reduce technical debt.

## Assertions and regressions

**Software testing** means checking behavior against an expectation. You state what must be true using an **assertion**. If the condition is not true, the test fails and tells you where and why.

- In Python the keyword is `assert`.
- Other languages use an assert keyword or macro, helpers with assert in the name (`assertThat`), or helpers that state something is expected or required (`expect`, `require`).

A **regression** is a new change breaking something that used to work. For example, a cart that once computed $108 for two $50 items with 8% tax returns `None` after a discount feature is added. Good tests catch that before it reaches production.

## The wider testing universe

"Does it work?" is only one question. Specialized kinds of testing address other qualities:

- **Performance testing**: behavior under a workload, such as 1,000 concurrent users.
- **Security testing**: vulnerabilities that could lead to breaches or unauthorized access.
- **Usability testing**: whether the application is intuitive and efficient for end users.
- Others: accessibility, compatibility, and more.

The foundation to build first is **automated functional testing**. Tests of different scopes (unit, integration, end-to-end) then combine into a **testing pyramid** and feed CI/CD pipelines.

## pytest: why and how to set it up

Python ships **unittest**, an xUnit-style framework that works well. The industry standard, however, is increasingly **pytest**:

- plain `assert` statements and much less boilerplate; unittest ideas transfer directly, you simply write less code;
- a rich plugin ecosystem and features such as **fixtures** and **parameterization**.

Setup: create and activate a virtual environment, install pytest with pip, then verify with `pytest --version`.

## Project structure and test discovery

Keep application code and tests in separate top-level directories, and add an `__init__.py` to each so both are **Python packages**. That keeps imports consistent, lets pytest resolve application modules from inside tests, and avoids `ModuleNotFoundError`. Without it, imports may fail depending on how pytest is executed; workarounds are the `-m` flag, `--import-mode=importlib`, or passing the working directory explicitly. The cleanest setup, used by most real-world projects, is `__init__.py` in every folder.

pytest **auto-discovers** tests by naming convention, with no extra setup:

- **Test files**: names start with `test_` or end with `_test.py`.
- **Test functions**: names start with `test_`.
- **Test classes** (optional): names start with `Test`; their methods start with `test_`.

These are pytest defaults and community conventions; `pytest.ini` can customize discovery if truly needed. Running `pytest` from the project root collects everything; you can also target a directory, a file or a single function.

## Reading test results

pytest prints one indicator per test:

- `.` passed
- `F` failed: an assertion was false
- `E` error: the test crashed before any assertion, for example a `RuntimeError`
- `s` skipped

Sections for errors and failures follow, then a short test summary of the totals. Diagnose in three steps:

1. **Failure indicator**: `F` or `E` plus the test name tell you *which* test did not pass.
2. **Error message**: the *why*. Assertion failures show `AssertionError` with expected versus actual values; other errors show the exception type (`TypeError`, `KeyError`) and a short description.
3. **Location**: the *where*. Use the file path and line number in the traceback; the test line is your starting point.

A failure is a precise bug report that arrives before the code leaves your machine, when fixing is cheapest.

## Quick reference

| Item | Meaning |
|---|---|
| `assert` | Python keyword that states what must be true; a false condition fails the test |
| Regression | A new change breaks something that used to work |
| `unittest` | Python's built-in xUnit-style test module |
| `pytest` | Modern standard runner: plain `assert`, less boilerplate, plugins, fixtures, parameterization |
| `pytest --version` | Verifies the installation and prints the version |
| `pytest` | Discovers and runs all tests from the project root |
| `test_*.py` / `*_test.py` | File names pytest collects automatically |
| `test_` prefix | Required for test functions and test-class methods |
| `Test` prefix | Required for optional test classes |
| `pytest.ini` | Customizes discovery when defaults do not fit |
| `__init__.py` | Marks a folder as a package so imports resolve consistently |
| `.` `F` `E` `s` | Passed, failed, error before assertion, skipped |

## Example

The snippets below apply package separation, convention-based discovery and reliable imports.

```jsx
my_project/│
├── my_app/│     └── __init__.py│     └── task_manager.py│     └──...(your application modules)│
├── tests/│     └── __init__.py│     └── test_example.py│     └──...(your test files)│
└──...
```

App and tests are separate packages, each with `__init__.py`; `test_example.py` matches the `test_` convention, so `pytest` finds it with no configuration.

```python
from my_app.task_managerimport add_task
```

A test importing app code through the package path; `__init__.py` in both folders is what makes this reliable.

⚠️ Note: the source writes `pipinstall pytest`, `pytest -m pytest`, `pytest.` and `task_managerimport` without spaces; these look like typos for `pip install pytest`, `python -m pytest`, `pytest .` and `task_manager import`.

## Remember

> `E` means the test crashed before any assertion ran; `F` means an assertion was false. Only the second one tells you the code disagreed with the expectation.

# TDD: Red, Green, Refactor

*Write a failing test, make it pass with minimal code, clean up, repeat.*

## In short

- **Test-Driven Development (TDD)** reverses the usual order: the test is written before the code it verifies.
- Every increment goes through three steps: **Red** (a failing test), **Green** (the minimum code that passes it), **Refactor** (better structure, same behavior).
- The loop is small, fast and repeated dozens of times in one session. Each pass adds a tiny, verified piece of functionality.
- A test must fail first. That proves the feature is really missing and proves the test itself can detect a change.
- Refactoring changes how the code is written, never what it does. The passing tests are the safety net.

## Why testing after the fact is not enough

Automated tests replace manual checking. Instead of running the program and clicking through features, tests cover the features and logic. Instead of hoping nothing broke after a change, the tests catch problems immediately. Manual testing is slow, inconsistent and misses subtle bugs.

Writing tests only after the code still leaves a gap. Testing later means debugging later: issues appear after a lot of code exists, so fixes are slow and disruptive. Without early, well-structured tests, code becomes **tightly coupled**, small changes feel risky, and **technical debt** builds up even though tests exist.

TDD was popularized by Kent Beck in the 1990s as part of **Extreme Programming (XP)**, which pushed good habits to the extreme: short cycles, constant refactoring, pair programming, continuous integration and self-documenting code. TDD keeps that spirit: small verified changes, less time lost to debugging and rework, and constant confidence in a safety net of tests.

## The cycle

TDD is not a checklist but a disciplined rhythm. For each new feature or fix:

1. **Red:** write a test for the next bit of behavior. It fails because the code does not exist yet.
2. **Green:** write the minimal code that makes the test pass.
3. **Refactor:** improve the implementation while the test keeps passing.

The cycle is designed to be very small and repeatable, not something done once a day. Because every new line of code is immediately tested, the chance of introducing bugs stays low.

## Red: write a failing test

This is the most counterintuitive step. The test is not written to confirm that existing code works. It is written for functionality that does not exist yet, or for a bug that has not been fixed. The failing test serves two purposes:

- **Proves what is missing:** it shows that the feature or fix is truly absent or broken.
- **Validates the test:** if the test does not fail when it should, it is not reliable and would give a false sense of security later. Failing first confirms the test setup can detect the coming change.

Test quality matters. A good test is:

- **Specific:** it covers one isolated piece of functionality or a single concept, so a failure points to one place. For example, `test_add_task_with_empty_title()` is specific; `test_tasks()` is too broad.
- **Descriptive:** the name states the scenario and the expected behavior, so a reader understands it without opening the test body. This self-documenting quality helps maintainability and team collaboration.

## Green: make it pass with the minimum

The goal is to turn the test green as quickly as possible with the absolute simplest code. This is not the moment for the most elegant, optimized or complete solution. It may mean hard-coding a return value for a specific input or adding a temporary, less elegant `if` statement.

This is not cheating. It validates the understanding of the problem and confirms that the test reads the code correctly. Doing the simplest thing gives rapid feedback, avoids **over-engineering** too early, and keeps the loop tight. The cleanup comes in the next step.

## Refactor: make the code right

**Refactoring** is restructuring existing code without changing its external behavior. With a passing test as a safety net, the internal design can be improved confidently: if something breaks, the test says so immediately. The rule is strict: external behavior must not change.

Refactoring is:

- Removing duplication.
- Improving the names of variables and functions.
- Simplifying complex or convoluted logic, extracting methods, or applying design patterns.

Refactoring is not:

- Adding new features or functionality.
- Changing behavior that users are accustomed to.
- Introducing new dependencies or external libraries.

The aim is always readability, maintainability and extensibility.

## Closing the loop

Each completed cycle delivers three things together: a feature that is fully implemented, protective tests that prove it works and guard it against regressions, and clean, readable, maintainable code. Repeating the loop gives faster feedback, a cleaner and more modular design, less debugging, and a codebase where every feature is covered as soon as it exists.

## Quick reference

| Term | Meaning |
| --- | --- |
| Test-Driven Development (TDD) | Write the test first, then the code that makes it pass. |
| Red | Write a failing test that defines the desired behavior. |
| Green | Write the minimum code needed to make that test pass. |
| Refactor | Improve internal structure without changing external behavior. |
| Refactoring | Restructuring existing code to make it easier to understand, maintain and extend. |
| Specific test | Covers one isolated piece of functionality or a single concept. |
| Descriptive test name | States the scenario and expected behavior without reading the test body. |
| `test_add_task_with_empty_title()` | Example of a specific, clearly named test. |
| `test_tasks()` | Example of a name that is too broad. |
| Safety net | The passing tests that reveal any behavior change during refactoring. |
| Extreme Programming (XP) | 1990s movement (Kent Beck) that took good habits to the extreme; origin of TDD. |
| Technical debt | Maintenance burden that builds up when code is tightly coupled and tested late. |

## Remember

> A test that has never failed has proven nothing, so watch it go red before you make it green.

# The Testing Pyramid

*Many fast, isolated tests at the base; fewer slow, realistic tests toward the top.*

## In short

- The **testing pyramid** has three layers: many **unit tests** at the base, fewer **integration tests** in the middle, a handful of **end-to-end (E2E) tests** at the top.
- Going up, tests get more realistic but slower, more fragile and more expensive to maintain.
- Unit tests check one function in isolation; integration tests check the seams between components; E2E tests replay a real user journey through the whole system.
- No layer is better than another. Each answers a different question; together they form a safety net.
- At every layer, test the flows that matter most, not every path.

## Why a pyramid

Each type of test has its own characteristics, costs and benefits. The pyramid keeps a broad base of cheap, fast tests that catch most issues quickly and a narrow top of expensive, slow ones reserved for the most critical checks.

When planning a strategy, ask which type of test gives the most value right now: fast feedback, proof that components connect, or confidence in the full user experience.

## Unit tests

A **unit test** checks the smallest testable piece of code, such as a single function or method. Its key property is **isolation**: no database, no network, no file system. It tests your code, not its environment, which is what lets it run in milliseconds and drive the rapid feedback loop of TDD.

Four characteristics define the layer:

- **Fast**: milliseconds per test, immediate feedback.
- **Isolated**: independent of external factors, so results are consistent and failures do not cascade.
- **Numerous**: each covers a tiny piece of functionality, so a project has hundreds or thousands.
- **Precise**: a failure points at the exact unit that broke.

Three angles cover a unit well:

1. **Happy path**: the ideal, most common scenario with no errors. It defines what success looks like and is usually the first test written for a feature. For example, adding an item increases the count.
2. **Edge cases**: inputs at or beyond the expected boundaries: minimums, maximums, empty values, odd combinations. Ask: what is the most unusual but still possible input? For example, an empty string as a title should raise a `ValueError`.
3. **Business rules**: the explicit policies the application must enforce, such as "no duplicates" or "an order must have a minimum value". For example, adding the same item twice leaves the count at one.

Tip: start with one function on the happy path and build from there.

## Integration tests

An **integration test** checks how components behave when plugged into each other: does the API trigger the right action in the logic layer, is data written to the database accurately, does one layer hand data to the next in the expected format?

Integration tests target the **seams** between architectural components and usually run against real dependencies instead of mocks. That makes them more realistic but slower and sometimes more brittle. The realism is the point: they catch issues that only appear when parts interact, especially **data contract** bugs where the data passed between components is not what the receiver expects.

Keep a balanced number: fewer than unit tests, more than E2E tests. Focus on the interactions that matter most to users.

## End-to-end tests

An **end-to-end test**, also called a **system test**, experiences the application as a user. The question is no longer "does this function return the right value?" but "can someone log in, add an item and check out without the app falling apart?" It is a dress rehearsal for the entire system: front end, back end, database and external services.

A typical flow:

1. **Launch the application** with all its services running.
2. **Navigate** by driving a real browser with a tool such as Selenium or Playwright.
3. **Authenticate** and confirm that protected areas become reachable.
4. **Access a feature**, checking navigation and routing.
5. **Create and verify** data, proving persistence and UI updates work together.

E2E tests confirm the system delivers on key user stories, but they are the **slowest** (minutes per run), **fragile** (renaming an element ID can break them even when the logic is correct) and **expensive** to write and maintain. Keep only a few, on the most critical user flows: registration, login, critical transactions, checkout.

## Quick reference

| Term | Meaning |
|---|---|
| Testing pyramid | Many unit tests, fewer integration tests, few E2E tests |
| Unit test | Tests one function or method in isolation, in milliseconds |
| Isolation | No database, network or file system in a unit test |
| Happy path | The expected scenario with no errors; usually the first test written |
| Edge case | Input at or beyond the boundary: empty, minimum, maximum, unusual |
| Business rule | An explicit policy the application must enforce |
| Integration test | Tests the seams between components against real dependencies |
| Data contract | The format and content one component expects from another |
| End-to-end (E2E) / system test | Replays a full user journey through the running system |
| Selenium / Playwright | Browser automation tools used to drive E2E tests |
| pytest fixture | Provides a resource such as a browser `driver` to a test |
| `conftest.py` | Shared pytest configuration applied to all tests without imports |

## Example

One task-management feature checked at all three layers applies isolation, seam testing and user-journey testing.

- **Unit**: add a task and assert the count increased; pass an empty title and expect a `ValueError`; add the same task twice and assert the count stays at one. No external dependency is touched.
- **Integration**: POST to `/api/tasks` with a new title, assert the API returns `201 Created`, then assert that `task_manager` registered the task. The second assertion proves the API talked to the logic layer across the seam.
- **End-to-end**: a pytest fixture provides a Selenium `driver` that opens a real browser. The test visits the login page, fills the username and password fields located by their IDs, clicks login, types a title into the new-task input, clicks add, and asserts that "My New Task" is visible in the task list. A changed element ID would break it.

## Remember

> A unit test that touches a database, the network or the file system is no longer a unit test, and every layer above it trades speed for realism.

# CI/CD with GitHub Actions

*Every push triggers an automated build and test run; passing code flows toward production.*

## In short

- **Continuous Integration (CI)** automatically builds and tests the code every time a change is pushed, so feedback arrives in minutes instead of at the end of the project.
- Integrating small changes into a shared repository often, ideally several times a day, is what prevents "integration hell".
- **Continuous Delivery** automates everything up to a release in staging and leaves the final approval to a human; **Continuous Deployment** also pushes to production automatically.
- A CI/CD pipeline has three steps: code commit, automated tests, release preparation, followed by deployment when practiced.
- In GitHub Actions the pipeline is a YAML file at `.github/workflows/ci.yml`, triggered by a push or a pull request.

## Why integrate continuously

Tests exist at several levels, from unit to integration to end to end, and they can be run by hand with pytest. The hard part is guaranteeing that all of them actually run on every change. Automating that is the point of CI.

The older model had developers working in isolation for days or weeks before merging. Problems surfaced only when many unmerged changes were finally combined, producing massive conflicts and bugs. This is **integration hell**. The later a bug is found, the more expensive it is to fix.

## What CI is

**Continuous Integration** is the practice of automatically building and testing the code whenever developers push changes. Its core principle is that developers integrate into a shared repository, such as Git, as often as possible.

- The integration itself is the trigger: each merge starts an automated build and the test suite.
- Feedback arrives within minutes. If a change breaks something, the team knows almost instantly.
- Because problems are caught while the change is still small and fresh, they are easier and cheaper to fix.
- The net effect is that teams move quickly and safely at the same time, with less risk and faster development.

## From CI to CD

**Continuous Delivery (CD)** and **Continuous Deployment** build on top of CI. Together the combined practice is called **CI/CD**: the fully automated journey of code from a developer's machine into production.

1. **Code commit**: a developer pushes changes to a version control system like Git. This action triggers the pipeline.
2. **Automated tests**: the pipeline runs the pre-configured suite of unit, integration and sometimes system tests to verify the quality and functionality of the new code.
3. **Preparing a release**: if all tests pass, the pipeline produces a deployable artifact. In continuous delivery this artifact is pushed to a staging or pre-production environment, and a human approves when to push it live.
4. **Continuous deployment**: the code is released to production automatically as soon as it passes the tests, so features and fixes reach users immediately.

The day-to-day loop is short: commit, the system builds, tests run, a report comes back. If something fails, fix it right away and commit again. CI keeps the code tested and stable; CD makes sure it reaches users quickly and safely.

## CI tools

- **GitHub Actions** and **GitLab CI/CD** are built directly into the repository.
- **CircleCI** is popular for its simplicity.
- **Jenkins** is one of the most powerful and customizable options.

## A workflow in GitHub Actions

A GitHub Actions pipeline is a **workflow**: a YAML file committed to the repository. Where the file lives is what matters. It must be under `.github/workflows/`, for example `.github/workflows/ci.yml`. The file name can change; the extension must be `yml`.

⚠️ Note: GitHub also accepts the `.yaml` extension for workflow files, so the "must be yml" statement is stricter than necessary.

A minimal workflow defines:

- A **name** for the workflow, shown in the Actions tab.
- The **trigger**: run whenever code is pushed or a pull request is opened, restricted to a branch. Changing the branch, say from `main` to `dev`, is a one-line edit.
- A **job** that runs on a machine, for example Ubuntu.
- The job's **steps**: check out the code, set up the runtime the project needs (Node.js, Python or Java), install dependencies, run the test script. For a JavaScript project the install step is `npm install`, followed by the test script.

Once committed, every push or pull request runs the workflow. Progress and results appear in the repository's **Actions** tab. A green check means all steps passed; the commit is ready to go.

## Quick reference

| Term | Meaning |
|---|---|
| Continuous Integration (CI) | Automatically build and test on every pushed change |
| Continuous Delivery | Automate up to a staging release; a human approves going live |
| Continuous Deployment | Automatically release to production once tests pass |
| CI/CD | The combined automated path from commit to production |
| Integration hell | Conflicts and bugs from merging many long-isolated changes at once |
| Pipeline | Commit, automated tests, release preparation, optional deployment |
| Deployable artifact | Build output produced when tests pass, pushed to staging or production |
| Workflow | A GitHub Actions pipeline defined in a YAML file |
| `.github/workflows/ci.yml` | Required location of a workflow file; extension must be `yml` |
| Trigger | Event that starts the workflow: push or pull request on a branch |
| Job / steps | Where the workflow runs (e.g. Ubuntu) and what it does: checkout, setup, install, test |
| Actions tab | Where workflow runs and their green check or failure are shown |

## Example

The example applies three concepts: the workflow file location, the push or pull request trigger, and the checkout, install, test sequence of steps.

```text
.github/workflows/ci.yml
```

This is the location rule. A workflow anywhere else never runs; the name before the `yml` extension is free to change.

```bash
npm install
```

This is the install-dependencies step, run on the Ubuntu job after the code is checked out and Node.js is set up. It is followed by the project's test script, which is what turns a push into a green check or a failure report.

## Remember

> The pipeline only exists if the workflow file sits under `.github/workflows/` with a `yml` extension; a correct workflow in the wrong place never triggers.

# Testing and AI

*Tests encode intent; AI supplies speed. Keep ownership of what matters.*

## In short

- AI assistants generate code fast, but they do not understand your business context, edge cases or unspoken requirements. Your tests do.
- Write the test first, then let AI generate the code that passes it. TDD is the safety net that makes AI speed safe.
- AI can generate tests, but auto-generated tests tend to mirror the current implementation, not the requirements. Generating tests after the code is code-first, test-later, not TDD.
- The sweet spot: you own critical paths and business logic; AI handles boilerplate and exploration.
- In CI/CD, AI helps with predictive test selection, flaky test detection, failure clustering and smart diagnostics on failure.

## Why AI makes TDD more critical

AI is changing how software is written. Coding assistants can produce large amounts of code quickly, which is genuinely useful for **boilerplate** and for solutions to common problems.

The limit is understanding. AI models are trained on vast, broad and sometimes biased datasets. They do not know your specific business rules, your edge cases or the requirements nobody wrote down. Speed without that knowledge produces code that looks right and may be wrong in exactly the places that matter.

This is why **Test-Driven Development (TDD)** becomes more important, not less. The engineering discipline of TDD lets you leverage the speed of AI while keeping a safety net in place.

## Tests as a living specification

A test can capture exactly the behavior and rules your application must satisfy. Written by a human who knows the context, it becomes a **living specification**: an executable statement of what the code should do, independent of how the code was produced or who produced it.

The practical workflow follows from this:

- Write the test first, encoding the requirement.
- Let AI generate the code that makes the test pass.
- Control of quality and intent stays with you, because the specification was yours.

The test does not care whether the implementation came from a person or a model. It only checks that the requirement holds.

## Using AI to generate tests

AI-powered tools can also work on the testing side. They can generate unit tests automatically or suggest additional test cases based on existing code. In end-to-end testing, some AI tools adapt to UI changes that would normally break traditional test scripts.

These capabilities do not replace testing fundamentals. The tempting shortcut is "AI writes the code and the tests, I just merge." This is risky for concrete reasons:

- **Auto-generated tests** usually match the current implementation rather than the requirements. If the code is wrong, the test confirms the wrong behavior.
- They can miss edge cases and business rules.
- They create a **false sense of safety** through shallow coverage.

Writing code first and then asking AI to produce tests for it is **code-first, test-later**. It is not TDD. Problems surface later, when fixes are slower, riskier and more disruptive. Designing meaningful tests and deciding what is worth validating remains a developer responsibility.

## The sweet spot: augment, do not delegate

The effective strategy is to use AI to complement your testing process rather than replace your judgment:

- Keep ownership of the critical paths and the business logic. Design those tests yourself.
- Let AI handle boilerplate tests and exploratory tests that expand coverage.
- Be intentional about where AI is used and where it is not.

The result is faster delivery without giving up the confidence that rigorous testing provides.

## AI in CI/CD pipelines

Beyond coding assistants, AI is valuable for optimizing continuous integration. Four common applications:

- **Predictive test selection**: AI analyzes the code changes and past test results to decide which tests matter most for a given commit. Instead of running the full suite, the pipeline runs a focused subset of the tests most likely to fail. Feedback arrives faster and failures are found sooner.
- **Flaky test detection**: AI notices tests that fail only sometimes. It spots the intermittent pattern across failing builds, tags the test as **flaky** and can **quarantine** it so it no longer blocks the build. The team chases real regressions instead of false alarms.
- **Failure clustering**: when many tests fail at once, AI groups the failures by stack trace or error signature, summarizes the details to speed up triage, and can suggest likely causes or fixes.
- **Smart diagnostics on failure**: AI collects the right logs, artifacts and screenshots, classifies the severity, and posts a concise update to the pull request or to Slack. It can also open issues automatically or prepare a pull request that reverts the recent changes, so the team can act immediately.

## Quick reference

| Term | Meaning |
|---|---|
| TDD | Write the test first, then the code that passes it; the safety net for AI-generated code |
| Living specification | Tests as an executable statement of what the code must do, regardless of who wrote the code |
| Boilerplate | Repetitive, common code; the area where AI generation is most useful |
| Auto-generated tests | Tests produced by AI from existing code; tend to mirror the implementation, not the requirements |
| Code-first, test-later | Writing code, then asking AI for tests; not TDD, problems found late |
| False sense of safety | Shallow coverage that looks reassuring but misses edge cases and business rules |
| Predictive test selection | AI picks the subset of tests most likely to fail for a commit instead of running all tests |
| Flaky test | A test that fails intermittently; AI tags and quarantines it so it does not block the build |
| Quarantine | Isolating a flaky test so its failures do not stop the pipeline |
| Failure clustering | Grouping failing tests by stack trace or error signature to speed up triage |
| Smart diagnostics | Auto-collecting logs, artifacts and screenshots, classifying severity, posting updates to PR or Slack |

## Remember

> An AI-generated test written after the code proves only that the code does what it does, not what it should do; the requirement has to come from you, first.

# Test-Driven Development in Practice

*Write the failing test first, pass it minimally, then refactor safely.*

## In short

- In TDD, tests drive the code: every line of implementation answers a test that failed first.
- The cycle is **red-green-refactor**: write a failing test, make it pass with minimal code, then improve the design.
- A good unit test checks one small unit in isolation, covering the happy path and the edge cases.
- Arrange, Act, Assert gives each test a clear beginning, middle and end.
- Refactoring changes structure, never behavior, and tests are the safety net that makes it safe.

## Unit tests and isolation

A **unit** is a small piece of code with one responsibility and no dependencies, for example a function that formats a first and last name.

A good **unit test** has three qualities:

- It checks behavior. Assertions compare the actual output with the exact expected output for given inputs.
- It runs in **isolation**. It needs no database, API or network, so it is self-contained, fast and reliable.
- It uses clear assertions. **pytest** lets you write plain `assert` statements.

Cover two kinds of cases. The **happy path** is the input that works as expected. The **edge case** is unusual input, such as empty strings or a negative amount.

## Arrange, Act, Assert

**Arrange, Act, Assert (AAA)** structures multi-step tests so the purpose is clear and failures are easy to diagnose. The framework does not enforce it; you choose it for readability.

- **Arrange**: prepare what the test needs, such as data, collaborators, test doubles or environment. Use **fixtures** to handle repetitive setup. Skip this step when there is nothing to prepare.
- **Act**: perform the single action under test, such as calling a function or method. If you are doing two things, write two tests.
- **Assert**: define success with a direct check so failures are obvious.

Multiple assertions are fine when they describe one outcome, for example status code 201 and the order ID after an API call. But pytest stops at the first failing assert, so split tests when you need independent failure signals.

## The red-green-refactor cycle

### Red

Write the test before the code exists. Name test files and test functions with the `test_` prefix, because pytest discovers them automatically. Keep tests in a separate file and name each one after the scenario and expected behavior.

The test fails, often with an `ImportError` because the function does not exist yet. This failure is a success: it proves the test works and tells you what to build next.

### Green

Write the minimum code that makes the test pass, even a hardcoded return value. Elegance does not matter yet. Reaching green quickly confirms the test is valid.

### Refactor

With a passing test, improve the code's structure without changing its behavior. Rerun the tests after every change. Green tests prove no regressions.

## Refactoring toward better design

A first step may replace a hardcoded result with real logic held in a **global variable**. That is quick and dirty: global state causes unexpected behavior, is hard to debug and prevents testing in isolation.

The standard fix is **encapsulation**: move data and logic into a class. Each instance keeps its own state, so each test can use a fresh instance without interference.

This change is itself driven by a test that shows the intended usage: create an instance in Arrange, call a method in Act, check an instance attribute in Assert. It fails against the old code until you refactor.

Refactoring never adds features; it keeps the code healthy over time.

## Growing the test suite

One scenario is not enough. Expand the suite deliberately:

- Add more complex happy paths, such as several inputs whose results must aggregate correctly.
- Add edge cases for invalid input and decide the expected behavior, such as raising a `ValueError`.
- Each new test starts red while existing ones stay green; then add just enough code to pass.

New features follow the same loop. Write a test describing the intended behavior, watch it fail, then write just enough code to pass. When the data model changes, a derived value can become a **property** computed from stored records, so it is always up to date.

## Quick reference

| Term | Meaning |
|---|---|
| Unit | Small piece of code with one job and no outside dependencies |
| Isolation | Testing a unit without databases, APIs or networks |
| Happy path | Input where everything works as expected |
| Edge case | Unusual or invalid input, such as empty or negative values |
| Red | Write a test that fails because the feature does not exist |
| Green | Write the minimum code to make the failing test pass |
| Refactor | Improve structure without changing behavior, tests stay green |
| Arrange | Set up data, collaborators and environment |
| Act | Perform one action under test |
| Assert | Check the outcome with a direct comparison |
| `test_` prefix | Naming convention pytest uses to discover tests |
| Fixture | Reusable setup that keeps Arrange small |

## Example

This example applies the red-green-refactor cycle, AAA and encapsulation to a small expense tracker, with application code in `tracker.py` and tests in `test_tracker.py`.

```bash
python -m pytest
```

- Red: a test calls `add_expense` before it exists, so the run fails with `ImportError`.
- Green: `add_expense` first just returns 10, the minimum that passes.
- Refactor: the hardcoded value becomes a running total in a global variable, then an `ExpenseTracker` class whose instance holds its own total. This is encapsulation replacing global state.
- Growing the suite: a test for multiple expenses checks aggregation, and a test for a negative amount expects a `ValueError`. An `if amount is less than or equal to zero` check turns it green.
- New feature: expenses become dictionaries with `amount` and `category` stored in `self.expenses`, `total` becomes a property that sums the amounts, and `get_expenses_by_category` returns only matching expenses.

## Remember

> A failing test in the red phase is a success: it proves the test works and tells you what to build next.

