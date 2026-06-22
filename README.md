# su26-ai301-contribution

## Practice Contribution Completed

Before selecting my main AI301 issue, I completed a practice contribution using the `firstcontributions/first-contributions` repository.

**Practice Repository:** https://github.com/firstcontributions/first-contributions  
**My Fork:** https://github.com/tokito-99/first-contributions  
**Practice Pull Request:** https://github.com/firstcontributions/first-contributions/pull/118779

For this practice contribution, I forked the repository, cloned it locally in VS Code, created a new branch, edited `Contributors.md`, committed my change, pushed the branch to GitHub, and opened a pull request. This helped me practice the standard open source workflow before working on my main AI301 contribution.

---

# Contribution 1: Expose Window Function Selection in `smooth_taubin`

**Contribution Number:** 1  
**Student:** Rishav Mishra  
**GitHub Username:** tokito-99  
**Issue:** https://github.com/pyvista/pyvista/issues/8695  
**Status:** Phase IV Complete - Pull Request Submitted; Awaiting Review  

---

## Issue Selection Update

My original selected issue for Week 1 was PyVista #8428: Inline Terminal Rendering for PyVista.

Original issue: https://github.com/pyvista/pyvista/issues/8428

After commenting on that issue, a maintainer replied that the feature had been mostly implemented in `pyvista-tui` and that contributions there were welcome. Because of that maintainer feedback, I treated the original issue as no longer the best fit for this contribution cycle and selected a more specific PyVista issue instead.

New selected issue: https://github.com/pyvista/pyvista/issues/8695

The new issue asks PyVista to expose window-function selection in `smooth_taubin()`, which is a smaller, more focused, and more testable contribution.

---

## Why I Chose This Issue

I chose this issue because it connects well with my background in Aerospace Engineering, CFD, mesh-based workflows, and Python-based engineering tools. PyVista is used for scientific visualization and mesh processing, so this issue is directly related to the type of computational and applied engineering software I am interested in contributing to.

This issue is also a good fit for this contribution cycle because it is specific and bounded. The current `smooth_taubin()` method wraps VTK's `vtkWindowedSincPolyDataFilter`, but it does not expose the filter's window-function selection options. The issue provides helpful context, including a concrete example where setting the window function to Nuttall fixed the reporter's smoothing issue. That makes the problem understandable and gives a clear direction for implementation.

---

## Understanding the Issue

### Problem Description

PyVista's `smooth_taubin()` method currently exposes several options from VTK's `vtkWindowedSincPolyDataFilter`, such as number of iterations, pass band, edge angle, feature angle, boundary smoothing, feature smoothing, non-manifold smoothing, and coordinate normalization. However, it does not expose the VTK filter's window-function selection options.

The issue reporter found that using the Nuttall window function solved their problem with non-manifold vertices moving during Taubin smoothing, but PyVista currently does not allow users to select that window function directly through the `smooth_taubin()` API.

### Expected Behavior

Users should be able to pass a window-function option into `smooth_taubin()`, for example:

```python
mesh.smooth_taubin(window_function="nuttall")
```

The selected string should map to the corresponding VTK method, such as `SetWindowFunctionToNuttall()`.

### Current Behavior

Before my implementation, calling `smooth_taubin()` with a `window_function` keyword failed because the method did not accept that parameter.

Example:

```python
mesh.smooth_taubin(window_function="nuttall")
```

Observed result before the fix:

```text
TypeError: smooth_taubin() got an unexpected keyword argument 'window_function'
```

### Affected Components

Likely affected files/components:

* `pyvista/core/filters/poly_data.py`
* The `PolyDataFilters.smooth_taubin()` method
* Existing tests that cover `smooth_taubin()`
* Documentation/docstring for `smooth_taubin()`

---

## Reproduction Process

### Environment Setup

I forked the PyVista repository and cloned my fork locally into VS Code.

Repository fork:

```text
https://github.com/tokito-99/pyvista
```

Local setup steps completed:

```powershell
cd C:\Users\miris\OneDrive\Desktop\Codepath
git clone https://github.com/tokito-99/pyvista.git
cd pyvista
git remote add upstream https://github.com/pyvista/pyvista.git
git switch -c feat/smooth-taubin-window-function
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e .
```

Setup challenge:

During the first installation attempt, the VTK dependency download was canceled before completion. I fixed this by creating and activating a virtual environment, then rerunning the editable PyVista install and allowing the VTK download to finish.

Successful verification:

```powershell
python -c "import pyvista as pv; print(pv.__version__)"
```

Output:

```text
0.49.dev0
```

I also located the relevant function using:

```powershell
git grep "def smooth_taubin"
```

Output:

```text
pyvista/core/filters/poly_data.py:    def smooth_taubin(  # noqa: PLR0917
```

### Steps to Reproduce

1. Activate the local PyVista development environment:

```powershell
.\.venv\Scripts\Activate.ps1
```

2. Confirm PyVista imports from the editable development install:

```powershell
python -c "import pyvista as pv; print(pv.__version__)"
```

3. Inspect the current `smooth_taubin()` signature:

```powershell
python -c "import pyvista as pv, inspect; mesh=pv.Sphere(); print(inspect.signature(mesh.smooth_taubin))"
```

4. Attempt to use the requested `window_function` argument:

```powershell
python -c "import pyvista as pv; mesh=pv.Sphere(); mesh.smooth_taubin(window_function='nuttall')"
```

5. Observed result before implementation:

```text
TypeError: smooth_taubin() got an unexpected keyword argument 'window_function'
```

6. Expected result:

The method should accept a valid window-function argument and apply the corresponding VTK window-function setting before running the smoothing algorithm.

### Reproduction Evidence

* **Working branch:** https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function
* **Screenshots/logs:** Local PowerShell output confirmed that PyVista installed successfully, imports as `0.49.dev0`, and that `smooth_taubin()` is located in `pyvista/core/filters/poly_data.py`.
* **My findings:** The issue was reproducible because the requested `window_function` keyword was not part of the `smooth_taubin()` API before implementation.

---

## Solution Approach

### Analysis

The root cause is that PyVista's `smooth_taubin()` method uses VTK's `vtkWindowedSincPolyDataFilter`, but only exposes some of that filter's options. The VTK filter supports selecting different window functions, but PyVista did not provide a user-facing argument to control that selection.

The issue reporter's workaround directly calls:

```python
alg.SetWindowFunctionToNuttall()
```

inside a copied version of the current PyVista implementation. A better PyVista solution avoids hard-coding Nuttall and instead exposes a validated `window_function` argument so users can choose among supported VTK options.

### Proposed Solution

I implemented a new optional `window_function` argument in `smooth_taubin()`.

The default behavior remains unchanged when `window_function` is not provided. When the argument is provided, PyVista validates the string and calls the correct VTK setter method.

Supported values implemented:

```text
"blackman"
"hamming"
"hanning"
"nuttall"
```

The user-facing values map to the VTK methods available in my local VTK installation:

* `blackman` -> `SetWindowFunctionToBlackman`
* `hamming` -> `SetWindowFunctionToHamming`
* `hanning` -> `SetWindowFunctionoHanning`
* `nuttall` -> `SetWindowFunctionToNuttall`

One important implementation detail is that VTK exposes the Hanning setter as `SetWindowFunctionoHanning`, so I verified the exact method name locally before implementing the mapping.

### Implementation Plan

Using UMPIRE framework adapted:

**Understand:**  
`smooth_taubin()` should expose VTK window-function selection because users currently cannot choose a window function through PyVista, even though the underlying VTK filter supports it.

**Match:**  
I looked for existing PyVista patterns for:

* optional string arguments
* input validation
* mapping user-facing strings to VTK setter methods
* filter tests in the core test suite
* docstring parameter documentation

**Plan:**

1. Modify `pyvista/core/filters/poly_data.py`.
2. Add a new optional `window_function` parameter to `smooth_taubin()`.
3. Keep the default behavior unchanged when `window_function=None`.
4. Add validation for supported string values.
5. Map valid values to the corresponding VTK methods.
6. Update the `smooth_taubin()` docstring with the new parameter and accepted values.
7. Add tests for valid window-function values.
8. Add a test for invalid window-function input.
9. Run the relevant tests before opening a PR.

**Implement:**  
Implementation is happening in Phase III on this branch:

```text
https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function
```

**Review:**  
I self-reviewed the change against PyVista's contribution expectations by checking that the change is scoped, the commits are atomic, the tests are targeted, and the diff only touches relevant files.

**Evaluate:**  
I verified the fix by:

* confirming the implementation accepts valid `window_function` values,
* confirming invalid values raise a clear `ValueError`,
* running existing and new tests related to `smooth_taubin()`,
* checking that existing/default `smooth_taubin()` behavior still works when `window_function` is not provided.

---

## Testing Strategy

### Unit Tests

* [x] Test that `smooth_taubin(window_function="nuttall")` runs without raising an unexpected keyword argument error.
* [x] Test all supported window-function strings:
  * `blackman`
  * `hamming`
  * `hanning`
  * `nuttall`
* [x] Test that an invalid `window_function` value raises a clear `ValueError`.
* [x] Test that the existing/default `smooth_taubin()` behavior still works when `window_function` is not provided.

### Integration Tests

* [x] Ran the existing PyVista test file that contains `smooth_taubin()` tests.
* [x] Confirmed the existing `smooth_taubin()` test still passes along with the new tests.

### Manual and Targeted Testing

I ran the targeted PyVista test command:

```powershell
python -m pytest tests/core/test_dataset_filters.py -k smooth_taubin
```

Final result:

```text
6 passed, 1 skipped, 777 deselected
```

This confirms that the existing `smooth_taubin()` behavior still passes, the new `window_function` tests pass, and the older-VTK compatibility test is skipped as expected on VTK 9.6.2.

### Testing Challenge Resolved

My first version of the new tests used `extract_geometry()`, but PyVista treats `extract_geometry()` as deprecated and the test failed. I updated the tests to use `extract_surface(algorithm=None)` instead, reran the targeted tests, and confirmed that all selected `smooth_taubin` tests passed.

---

## Implementation Notes

### Week 1 Progress

I originally selected PyVista #8428, which proposed inline terminal rendering for PyVista plots. After maintainer feedback indicated that the feature was mostly implemented in `pyvista-tui`, I pivoted to a more specific and bounded PyVista issue.

### Week 2 Progress

I selected PyVista #8695 as the new contribution issue. I forked PyVista, cloned the fork locally, added the upstream remote, created a working branch, created a virtual environment, installed PyVista in editable mode, verified the local development version, and located the relevant `smooth_taubin()` implementation.

### Week 3 Progress

**What I built:**

* Implemented the core fix for PyVista issue #8695.
* Added a new optional `window_function` argument to `smooth_taubin()` in `pyvista/core/filters/poly_data.py`.
* Preserved the existing default behavior when `window_function` is not provided.
* Added validation for supported window-function values.
* Mapped the supported user-facing string values to the corresponding VTK setter methods:
  * `blackman` -> `SetWindowFunctionToBlackman`
  * `hamming` -> `SetWindowFunctionToHamming`
  * `hanning` -> `SetWindowFunctionoHanning`
  * `nuttall` -> `SetWindowFunctionToNuttall`
* Added tests for valid and invalid `window_function` inputs in the existing PyVista test file.
* Updated the `smooth_taubin()` docstring to document the new parameter and accepted values.

**How I confirmed the changes were scoped and atomic:**

* I split the initial implementation into three separate commits instead of one large commit:
  * `b7c8257ff` - Add window_function option to smooth_taubin
  * `72f221fb4` - Test smooth_taubin window_function handling
  * `8da4af8fa` - Document smooth_taubin window_function
* I ran `git log --oneline upstream/main..HEAD` to confirm the branch contains only focused commits.
* I ran `git diff --stat upstream/main...HEAD` to confirm that only two relevant files changed.
* After the Week 4 compatibility update, the final diff showed:
  * `pyvista/core/filters/poly_data.py` - 29 insertions
  * `tests/core/test_dataset_filters.py` - 25 insertions
  * Total: 2 files changed, 54 insertions
* I ran `git diff upstream/main...HEAD` to inspect the actual code changes and confirm there were no unrelated files, debug prints, or broad refactors.

**Testing performed:**

```powershell
python -m pytest tests/core/test_dataset_filters.py -k smooth_taubin
```

Result:

```text
6 passed, 1 skipped, 777 deselected
```

**Branch:**

https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function

### Week 4 Progress

**Pre-submission review and compatibility work:**

* Audited the implementation against PyVista's supported VTK test matrix before opening a pull request.
* Found that VTK's window-function setter methods are available in VTK 9.4.0 and later, while PyVista also tests older VTK versions.
* Added an explicit `VTKVersionError` for users who request `window_function` with VTK older than 9.4.0.
* Added test markers for VTK 9.4 or newer and a separate test for the older-VTK error path.
* Updated the docstring to state that window-function selection requires VTK 9.4.0 or later.
* Added a fourth focused compatibility commit:
  * `fb6681903` - Handle smooth_taubin window functions on older VTK
* Tested the focused `smooth_taubin` cases with multiple VTK versions:
  * VTK 9.3.1: 2 passed, 5 skipped.
  * VTK 9.4.2: 6 passed, 1 skipped.
  * VTK 9.6.2: 6 passed, 1 skipped.
* Ran scoped pre-commit checks, including codespell, docstring validation, Ruff formatting, and Ruff linting. All applicable checks passed.
* Rebased the branch onto the latest `upstream/main`, reran the focused tests and pre-commit checks, and safely updated the remote feature branch with `--force-with-lease`.

**Current Week 4 status:**

Pull request [pyvista/pyvista#8779](https://github.com/pyvista/pyvista/pull/8779) has been submitted to merge the feature branch into PyVista's `main` branch. The PR is open and awaiting maintainer review. Its GitHub Actions workflows are also awaiting approval from a PyVista maintainer because the contribution comes from a fork.

### Code Changes

* **Files modified:**
  * `pyvista/core/filters/poly_data.py`
  * `tests/core/test_dataset_filters.py`

* **Key commits:**
  * `b7c8257ff` - Add window_function option to smooth_taubin
  * `72f221fb4` - Test smooth_taubin window_function handling
  * `8da4af8fa` - Document smooth_taubin window_function
  * `fb6681903` - Handle smooth_taubin window functions on older VTK

* **Approach decisions:**
  * I exposed a general `window_function` argument instead of hard-coding only the Nuttall window function.
  * I kept the default behavior unchanged when `window_function` is not provided.
  * I added validation so unsupported values raise a clear `ValueError`.
  * I kept the change scoped to the smoothing filter implementation and its existing test file.

---

## Pull Request

**PR Link:** [pyvista/pyvista#8779](https://github.com/pyvista/pyvista/pull/8779)

**PR Description:** Exposes VTK window-function selection through PyVista's `smooth_taubin()` API. The change supports Blackman, Hamming, Hanning, and Nuttall window functions, validates invalid input, preserves existing default behavior, and provides a clear compatibility error when VTK is older than 9.4.0.

**Maintainer Feedback:**

* Maintainer feedback on original issue #8428 indicated that the terminal rendering feature was mostly implemented in `pyvista-tui`, so I selected a new PyVista issue for this contribution.
* On issue #8695, maintainer `@akaszynski` confirmed that the issue was still available and invited the contribution.
* Pull request #8779 was submitted on June 22, 2026. No review feedback has been received yet, and the PR is awaiting maintainer review and workflow approval.

**Status:** Awaiting review

**Phase IV Status:** Complete - pull request submitted

---

## Learnings & Reflections

### Technical Skills Gained

I learned how to set up a large scientific Python project locally using a fork, upstream remote, virtual environment, and editable installation. I also practiced tracing an issue from a GitHub feature request to a specific function in the codebase.

During Week 3, I gained more experience working inside a mature scientific Python codebase. I practiced making a small API extension, validating user-facing string input, mapping PyVista-level options to underlying VTK methods, writing targeted tests in an existing test file, and keeping implementation, tests, and documentation in separate atomic commits.

### Challenges Overcome

The first PyVista issue I selected was not ideal after maintainer feedback because the feature had mostly been implemented in a related project. I pivoted to a more focused issue while staying within the PyVista ecosystem. During setup, I also resolved an installation interruption caused by canceling the VTK dependency download.

During Week 3, my first test version used `extract_geometry()`, but PyVista treated that as deprecated and the test run failed. I corrected the tests to use `extract_surface(algorithm=None)` instead. I also had to verify the exact VTK method names locally because the Hanning method is exposed as `SetWindowFunctionoHanning`.

### What I'd Do Differently Next Time

I would wait for maintainer feedback before fully committing to an issue in my README. I would also create a virtual environment before installing dependencies to avoid installing packages into the global/user Python environment.

For future implementation phases, I would also check for deprecated helper methods before using them in new tests, and I would inspect nearby tests more carefully before adding new test cases.

---

## Resources Used

* https://github.com/pyvista/pyvista/issues/8428
* https://github.com/pyvista/pyvista/issues/8695
* https://github.com/pyvista/pyvista
* https://github.com/pyvista/pyvista/blob/main/CONTRIBUTING.rst
* https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function
