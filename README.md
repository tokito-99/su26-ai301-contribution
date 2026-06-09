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

# Contribution 1: Inline Terminal Rendering for PyVista

**Contribution Number:** 1
**Student:** Rishav Mishra
**Issue:** https://github.com/pyvista/pyvista/issues/8428
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it connects well with my background in Aerospace Engineering, CFD, and Python-based engineering visualization workflows. PyVista is used for 3D visualization and mesh analysis, which makes this issue relevant to scientific computing, simulation, and engineering software. Since I have experience with CFD workflows and visualization, this issue feels meaningful and connected to my technical interests.

This issue also seems like a good fit for a first structured open-source contribution because it has a specific feature request, clear motivation, and concrete possible integration points. The issue describes a practical problem: PyVista plots currently do not render directly inside terminal output, which can be inconvenient for SSH sessions, CI/CD workflows, and IPython terminal users. I am interested in learning how plotting behavior is handled inside a mature open-source Python project and how a feature can be added in a careful, maintainable way.

---

## Understanding the Issue

### Problem Description

PyVista currently supports plotting meshes, but it does not have a built-in way to display rendered mesh images directly inline inside capable terminals. When a user calls `mesh.plot()` or uses plotting from a terminal workflow, the plot usually opens in a GUI window or must be handled through screenshots/notebook-specific workflows. The requested feature is to allow PyVista to render a static image of the mesh directly into the terminal output using terminal image protocols.

### Expected Behavior

When terminal rendering is requested, PyVista should render the scene off-screen, capture the rendered image, and emit that image directly to the terminal using a supported image protocol. The issue specifically mentions iTerm2 inline images, Sixel-capable terminals, and possible fallback rendering using `textual-image`.

### Current Behavior

At the moment, PyVista does not automatically display rendered mesh output inline in the terminal. In off-screen rendering workflows, a plot may be created and disposed without visible output unless the user explicitly saves or requests a screenshot. This limits usability in SSH sessions, terminal-based workflows, and CI/CD environments where opening a GUI window may be unavailable or disruptive.

### Affected Components

Based on the issue description, the likely affected components are:

* `Plotter.show()`
* `DataSet.plot()`
* PyVista plotting utilities
* The existing `pyvista render` CLI
* Possible future IPython terminal display hooks

For an initial contribution, I do not plan to implement every integration point at once. A smaller and more reviewable first approach would likely focus on an explicit opt-in path, possibly through `Plotter.show()`, while asking maintainers what API design they prefer.

---

## Reproduction Process

### Environment Setup

TBD in Phase II.

I will fork and clone the PyVista repository, set up the development environment, and verify that I can run a basic PyVista plotting example locally. I will also review PyVista's contribution instructions before making implementation changes.

### Steps to Reproduce

1. Install and run PyVista locally.
2. Create or load a sample mesh, such as a PyVista example mesh.
3. Run a plotting command from a terminal workflow, such as `mesh.plot()`.
4. Observe that the rendered mesh does not appear inline in the terminal output.

### Reproduction Evidence

* **Commit showing reproduction:** TBD
* **Screenshots/logs:** TBD
* **My findings:** TBD

---

## Solution Approach

### Analysis

The issue appears to require a bridge between PyVista's existing off-screen rendering capability and terminal image display protocols. The proposed idea in the issue is to render off-screen, capture the framebuffer as an image, and then send the image to the terminal using a supported terminal protocol.

One important design concern is that automatic terminal rendering could accidentally trigger in cases where it should not, such as during tests or commands like `pv.GPUInfo()`. Because of that, I think the safest initial direction is to explore an explicit opt-in argument instead of assuming that every off-screen render should appear in the terminal.

### Proposed Solution

My initial proposed solution is to investigate a small opt-in implementation path for terminal rendering. A possible first implementation could add an explicit argument such as `terminal` or `show_terminal` to one plotting pathway, likely starting with `Plotter.show()`. This would keep the first pull request focused and avoid changing every plotting entry point at once.

The initial version may prioritize the iTerm2 inline image path because the issue notes that iTerm2 can work using the standard library. Broader fallback support through `textual-image` could be treated as a later step or optional dependency, depending on maintainer guidance.

### Implementation Plan

Using UMPIRE framework adapted:

**Understand:**
PyVista needs a way to display a rendered mesh image directly in capable terminal output instead of only opening a GUI window or requiring explicit screenshot handling.

**Match:**
I will look for existing PyVista code paths related to `Plotter.show()`, screenshots, off-screen rendering, CLI rendering, and plotting tests. I will also look for existing patterns in PyVista for optional dependencies and explicit plotting keyword arguments.

**Plan:**

1. Set up the PyVista development environment locally.
2. Reproduce the current terminal behavior with a simple mesh plotting example.
3. Locate the relevant plotting code for `Plotter.show()` and screenshot generation.
4. Investigate where an explicit terminal rendering option could be added with minimal disruption.
5. Ask maintainers for guidance on the preferred API name and scope.
6. Implement the smallest reviewable change.
7. Add or update tests where possible.
8. Run formatting, linting, and relevant plotting tests before submitting a PR.

**Implement:**
TBD in Phase III. Branch and commit links will be added after development starts.

**Review:**
Before opening a PR, I will check that the change is narrow, documented, tested, and aligned with PyVista's contribution guidelines.

**Evaluate:**
I will verify the change with a simple PyVista mesh example and run the relevant test suite. If terminal protocol behavior is difficult to test directly, I will test the output-generation logic separately from the actual terminal display.

---

## Testing Strategy

### Unit Tests

* [ ] Test that terminal rendering is only triggered when explicitly requested.
* [ ] Test that the terminal display helper receives or generates image data correctly.
* [ ] Test that normal `Plotter.show()` behavior is unchanged when terminal rendering is not requested.

### Integration Tests

* [ ] Run a basic mesh plotting workflow with the new terminal option enabled.
* [ ] Run existing plotting tests to ensure normal plotting behavior is not broken.

### Manual Testing

TBD in Phase II/III.

Potential manual tests include running a simple PyVista mesh example from a supported terminal and verifying that the image appears inline when the terminal rendering option is enabled.

---

## Implementation Notes

### Week 1 Progress

I selected PyVista issue #8428 as my first contribution issue. I reviewed the issue description, confirmed that it is related to inline terminal rendering for PyVista plots, and identified the main proposed integration points: `Plotter.show()`, `DataSet.plot()`, the `pyvista render` CLI, and possible IPython display hooks.

I also completed a practice open-source contribution using the `firstcontributions/first-contributions` repository. That practice contribution helped me learn the basic workflow of forking a repository, cloning it locally, creating a branch, making a change, committing, pushing, and opening a pull request.

### Code Changes

* **Files modified:** TBD
* **Key commits:** TBD
* **Approach decisions:** I will start by focusing on a small, explicit opt-in implementation rather than trying to implement the full terminal rendering feature across every possible integration point.

---

## Pull Request

**PR Link:** TBD

**PR Description:** TBD

**Maintainer Feedback:**

* TBD

**Status:** Not submitted yet

---

## Learnings & Reflections

### Technical Skills Gained

So far, I have learned how to evaluate an open-source issue for feasibility, check whether it is active and claimable, and connect the issue to my technical background and learning goals. I also practiced the GitHub contribution workflow through a practice pull request.

### Challenges Overcome

During the practice contribution, I encountered a GitHub authentication issue because Git was initially connected to a different GitHub account. I resolved it by re-authenticating with the correct account and successfully pushed my branch.

### What I'd Do Differently Next Time

Next time, I would check my GitHub authentication and remote repository settings before pushing. For the PyVista issue, I will also avoid making assumptions about the final API design until I receive maintainer guidance.

---

## Resources Used

* https://github.com/pyvista/pyvista/issues/8428
* https://github.com/pyvista/pyvista
* https://github.com/pyvista/pyvista/blob/main/CONTRIBUTING.rst
* https://github.com/firstcontributions/first-contributions
* https://github.com/firstcontributions/first-contributions/pull/118779
