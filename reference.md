# Reference

This document provides reference information for the `gcp-scratch` repository.

## Scripts

### `push.sh`

This script simplifies the process of adding, committing, and pushing changes to the repository. It automates the following steps:
- Exits immediately if any command fails.
- Verifies that a commit message is provided.
- Stages all new, modified, and deleted files.
- Commits the changes with the provided message.
- Pushes the changes to the remote repository.

**Usage:**

```bash
./push.sh "Your commit message"
```

If no commit message is provided, the script will exit with an error.

## Functions

### `add(a: int, b: int) -> int`

Adds two numbers.

**Note:** The current implementation is `a + b * 2`, which is not a standard addition.

**Parameters:**

*   `a` (int): The first number.
*   `b` (int): The second number.

**Returns:**

*   `int`: The sum of the two numbers, with the second number doubled.

### `simulate_fibonacci(n: int) -> list[int]`

The `simulate_fibonacci` function is now called with an argument of `1234` instead of `10` in the main execution block.
