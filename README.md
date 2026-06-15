# su26-ai301-contribution
# Contribution 1: TBD - Issue Not Selected Yet

**Contribution Number:** 1
**Student:** Rishav Mishra
**GitHub Username:** tokito-99
**Issue:** TBD
**Status:** Phase I - In Progress

---

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
**Issue:** https://github.com/pyvista/pyvista/issues/8695
**Status:** Phase II Complete

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

Calling `smooth_taubin()` with a `window_function` keyword currently fails because the method does not accept that parameter.

Example:

```python
mesh.smooth_taubin(window_function="nuttall")
```

Current result:

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

5. Observed result:

```text
TypeError: smooth_taubin() got an unexpected keyword argument 'window_function'
```

6. Expected result:

The method should accept a valid window-function argument and apply the corresponding VTK window-function setting before running the smoothing algorithm.

### Reproduction Evidence

* **Working branch:** https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function
* **Screenshots/logs:** Local PowerShell output confirms that PyVista installed successfully, imports as `0.49.dev0`, and that `smooth_taubin()` is located in `pyvista/core/filters/poly_data.py`.
* **My findings:** The issue is reproducible because the requested `window_function` keyword is not currently part of the `smooth_taubin()` API.

---

## Solution Approach

### Analysis

The root cause is that PyVista's `smooth_taubin()` method uses VTK's `vtkWindowedSincPolyDataFilter`, but only exposes some of that filter's options. The VTK filter supports selecting different window functions, but PyVista currently does not provide a user-facing argument to control that selection.

The issue reporter's workaround directly calls:

```python
alg.SetWindowFunctionToNuttall()
```

inside a copied version of the current PyVista implementation. A better PyVista solution would avoid hard-coding Nuttall and instead expose a validated `window_function` argument so users can choose among supported VTK options.

### Proposed Solution

I plan to add a new optional `window_function` argument to `smooth_taubin()`.

The default behavior should remain unchanged when `window_function` is not provided. When the argument is provided, PyVista should validate the string and call the correct VTK setter method.

Possible accepted values:

```text
"nuttall"
"blackman"
"hanning"
"hamming"
```

The final set of accepted values should match the VTK methods available in the installed VTK version and PyVista's API style.

### Implementation Plan

Using UMPIRE framework adapted:

**Understand:**
`smooth_taubin()` should expose VTK window-function selection because users currently cannot choose a window function through PyVista, even though the underlying VTK filter supports it.

**Match:**
I will look for existing PyVista patterns for:

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
6. Update the `smooth_taubin()` docstring with the new parameter and a short explanation.
7. Add tests for valid window-function values.
8. Add a test for invalid window-function input.
9. Run the relevant tests before opening a PR.

**Implement:**
Implementation will happen in Phase III on this branch:

```text
https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function
```

**Review:**
I will self-review the change against PyVista's contribution guidelines, especially API style, docstring style, validation style, and test expectations.

**Evaluate:**
I will verify the fix by:

* confirming `mesh.smooth_taubin(window_function="nuttall")` no longer raises an unexpected keyword argument error
* confirming invalid values raise a clear error
* running existing and new tests related to `smooth_taubin()`
* checking that existing default behavior is unchanged when `window_function` is not provided

---

## Testing Strategy

### Unit Tests

* [ ] Test that `smooth_taubin(window_function="nuttall")` runs without raising an unexpected keyword argument error.
* [ ] Test other supported window-function strings if supported by the installed VTK version.
* [ ] Test that an invalid `window_function` value raises a clear validation error.
* [ ] Test that the default behavior still works when `window_function` is not provided.

### Integration Tests

* [ ] Run the existing test file that contains `smooth_taubin()` tests.
* [ ] Run relevant PyVista core filter tests to confirm the new argument does not break existing filter behavior.

### Manual Testing

Manual test command planned after implementation:

```powershell
python -c "import pyvista as pv; mesh=pv.Sphere(); smoothed=mesh.smooth_taubin(window_function='nuttall'); print(smoothed.n_points)"
```

Expected result after implementation:

```text
The command should complete successfully and print the number of points in the smoothed mesh.
```

---

## Implementation Notes

### Week 1 Progress

I originally selected PyVista #8428, which proposed inline terminal rendering for PyVista plots. After maintainer feedback indicated that the feature was mostly implemented in `pyvista-tui`, I pivoted to a more specific and bounded PyVista issue.

### Week 2 Progress

I selected PyVista #8695 as the new contribution issue. I forked PyVista, cloned the fork locally, added the upstream remote, created a working branch, created a virtual environment, installed PyVista in editable mode, verified the local development version, and located the relevant `smooth_taubin()` implementation.

### Code Changes

* **Files modified:** No implementation files modified yet.
* **Key commits:** TBD in Phase III.
* **Approach decisions:** I plan to expose a general `window_function` argument instead of hard-coding only the Nuttall window function. This should make the fix more flexible while keeping the default behavior unchanged.

---

## Pull Request

**PR Link:** TBD

**PR Description:** TBD

**Maintainer Feedback:**

* Maintainer feedback on original issue #8428 indicated that the terminal rendering feature was mostly implemented in `pyvista-tui`, so I selected a new PyVista issue for this contribution.
* Awaiting/monitoring feedback on #8695 before opening a PR.

**Status:** Not submitted yet

---

## Learnings & Reflections

### Technical Skills Gained

I learned how to set up a large scientific Python project locally using a fork, upstream remote, virtual environment, and editable installation. I also practiced tracing an issue from a GitHub feature request to a specific function in the codebase.

### Challenges Overcome

The first PyVista issue I selected was not ideal after maintainer feedback because the feature had mostly been implemented in a related project. I pivoted to a more focused issue while staying within the PyVista ecosystem. During setup, I also resolved an installation interruption caused by canceling the VTK dependency download.

### What I'd Do Differently Next Time

I would wait for maintainer feedback before fully committing to an issue in my README. I would also create a virtual environment before installing dependencies to avoid installing packages into the global/user Python environment.

---

## Resources Used

* https://github.com/pyvista/pyvista/issues/8428
* https://github.com/pyvista/pyvista/issues/8695
* https://github.com/pyvista/pyvista
* https://github.com/pyvista/pyvista/blob/main/CONTRIBUTING.rst
* https://github.com/tokito-99/pyvista/tree/feat/smooth-taubin-window-function

