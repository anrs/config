# Go Code Style Guidelines

This document outlines the Go code style preferences and conventions adopted for this project. Adherence to these guidelines ensures consistency, readability, and maintainability across the codebase.

## General Principles

*   **Clarity and Readability:** Code must be easily understandable and maintainable.
*   **Idiomatic Go:** Follow established Go patterns and best practices.
*   **Descriptive Naming:** Employ clear, concise, and descriptive names for packages, types, functions, and variables.

## Specific Conventions

### 1. Package and Imports

*   **`package main`:** Reserved for executable programs.
*   **Import Grouping:** Imports are organized into distinct groups, separated by a blank line:
    1.  Standard library packages (e.g., `context`, `fmt`, `os`).
    2.  Third-party packages (e.g., `gitlab.agodadev.io/...`, `k8s.io/...`, `sigs.k8s.io/...`).
*   **Import Aliases:** Use aliases judiciously for long or conflicting package names (e.g., `utilruntime "k8s.io/apimachinery/pkg/util/runtime"`).

### 2. Constants and Global Variables

*   **Global `scheme`:** Declare global `scheme` variables for Kubernetes API registration.
*   **`const` for Literals:** Use global `const` blocks to define repeated string literals or magic numbers to improve maintainability and avoid duplication.
*   **`init()` for Scheme Registration:** Utilize `init()` functions to register API schemes, often paired with `utilruntime.Must` for handling unrecoverable initialization errors.

### 3. Function Structure and Ordering

*   **`main()` Function:** Provides a high-level overview of the program's execution flow, encompassing flag parsing, client initialization, resource fetching, and output.
*   **Helper Functions:** Employ small, focused helper functions (e.g., `getName`, `getObjByName`) to encapsulate common operations, thereby improving code readability and conciseness.
*   **Function Placement:** Helper functions or functions called by others should generally be defined after their callers within the same file.

### 4. Variable Naming

*   **Descriptive Names:** Use full, descriptive names for variables (e.g., `managementContext`, `workloadClusterName`, `fqdn`).
*   **Shorter Local Names:** For local variables, shorter, context-appropriate names are acceptable (e.g., `kc` for `KubernetesCluster`).

### 5. Error Handling

*   **Multiple Return Values:** Functions must return errors as the second return value (`(..., error)`).
*   **Error Wrapping:** Use `fmt.Errorf("...: %w", err)` to wrap errors, preserving the original error context.
*   **Immediate Checking:** Check for errors immediately after function calls using `if err != nil`.
*   **`os.Exit(1)`:** For critical, unrecoverable errors in `main`, print the error message using `fmt.Printf` and then exit with `os.Exit(1)`.

### 6. Logging and Output

*   **`fmt.Printf` for CLI Tools:** In command-line applications, use `fmt.Printf` to provide clear, informative output about the script's progress. This includes logging actions being taken (e.g., "Fetching Service..."), successful operations, and detailed error messages.
*   **No External Logging Library:** The standard `fmt` package is preferred unless a specific logging framework is explicitly introduced.

### 7. Asynchronous Waiting and Polling

*   **Polling with Timeout:** When waiting for a change in the cluster state (e.g., a resource update or deletion), use a polling mechanism.
*   **Pattern:** Implement this using a `for` loop with a `select` statement.
    *   Use a `time.Ticker` for periodic checks.
    *   Use a `context.WithTimeout` to create a context that will cancel the operation after a deadline, preventing the script from hanging indefinitely.
    *   Always `defer` the `cancel()` function immediately after creating the timeout context.

### 8. Context Passing

*   **Explicit Context**: Pass `context.Context` as the first argument to functions performing I/O, API calls, or other long-running tasks.
*   **Lifecycle Management**: Use `context.WithCancel` in `main` or other entry points to manage the lifecycle of the application's operations. Always call the returned `cancel` function, typically with `defer`.

### 9. Kubernetes Client Usage

*   **`clientcmd`:** Employ for kubeconfig loading and context handling.
*   **`sigs.k8s.io/controller-runtime/pkg/client`:** Use for programmatic interaction with Kubernetes API objects.
*   **`context.Background()`:** Acceptable for simple, fire-and-forget operations or as a root for new contexts, but prefer passing contexts down from `main`.

### 10. Flag Parsing

*   **Standard `flag` Package:** Utilize the built-in `flag` package for command-line argument parsing.