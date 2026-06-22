# CI Pipeline Failure Diagnosis

## Failure 1: Dependency Install Step

**Step Name:** Install Dependencies

**Error Message (from logs):**

```
npm ci can only install packages when your package.json and package-lock.json are in sync
Missing: lodash@4.18.1 from lock file
```

**Explanation:**
The CI pipeline is using `npm ci`, which requires `package.json` and `package-lock.json` to be perfectly synchronized. The error shows that `lodash@4.18.1` exists in `package.json` but is missing from `package-lock.json`.

Because of this mismatch, `npm ci` fails and stops the pipeline before any tests or build steps can run. The root cause is that the lockfile was not updated and committed after dependency changes, making the install process non-reproducible.

---

## Failure 2: Incorrect Unit Test Assertion

**Step Name:** Run Tests (Jest)

**Error Message (from logs):**

```
Expected: 100
Received: 90

expect(received).toBe(expected)
```

**Location:**
`src/payments/calculateDiscount.test.js`

**Explanation:**
The test expects `calculateDiscount(100, 10)` to return `100`, but the actual output is `90`. This shows that the function is correctly applying a 10% discount (100 → 90).

Therefore, the implementation is correct and the issue lies in the test assertion. The expected value in the test is incorrect and does not match the intended behavior of the function.

---

## Failure 3: Incorrect Matcher for Object Comparison

**Step Name:** Run Tests (Jest)

**Error Message (from logs):**

```
If it should pass with deep equality, replace "toBe" with "toStrictEqual"
```

**Location:**
`src/utils/formatCurrency.test.js`

**Explanation:**
The test uses `toBe()` to compare two objects returned by `formatCurrency()`. However, `toBe()` checks reference equality (same memory location), not deep structural equality.

Since the function returns a new object, the test fails even though the values match. The correct approach is to use `toEqual()` or `toStrictEqual()` for deep comparison of object properties.

---

## Failure 4: Workflow Sequencing / Missing Dependency Setup

**Step Name:** Test Job (GitHub Actions Workflow)

**Error Message (from logs):**

```
Error: Cannot find module 'jest'
```

**Explanation:**
The test step is executing before the required dependencies are properly installed. This indicates a sequencing issue in the GitHub Actions workflow.

The workflow is missing proper job ordering or setup steps such as:

* Ensuring `actions/checkout` runs before anything else
* Installing dependencies before running tests
* Possibly missing a `needs:` dependency between jobs

As a result, Jest is not available in the environment when the test command runs, causing the failure.

---

## Summary

The CI pipeline is failing due to three main issues:

1. Dependency lockfile mismatch preventing `npm ci` from installing packages
2. Incorrect test assertion expecting the wrong output value
3. Incorrect matcher used for object comparison in tests
4. Workflow misconfiguration causing tests to run before dependencies are installed

Fixing these issues will restore a fully green CI pipeline.
