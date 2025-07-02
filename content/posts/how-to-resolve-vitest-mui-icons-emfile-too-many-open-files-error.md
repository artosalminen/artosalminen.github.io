---
title: "How to Resolve Vitest `EMFILE: too many open files` Errors on a High Thread Count Machine"
date: 2025-07-02
series: ["React", "coding", "Vite", "Vitest"]
weight: 1
tags: ["React", "coding"]
author: "Arto Salminen"
---

If you're running your Vitest suite on a high thread count machine—like an 8-core or 16-core development machine—you may encounter a frustrating error that looks like this:

```
EMFILE: too many open files
```

This issue can be cryptic at first, especially when your system doesn't seem anywhere near running out of resources. But the root cause might not be in your system limits—it could be in your import statements.

## The Unexpected Culprit: Named Imports from MUI

In my case, after hours of digging, I discovered that the error was caused by using **named imports** from Material UI (MUI) components and icons. These named imports, when used across many test files, were triggering Vitest to open an excessive number of files behind the scenes.

Why? Because a named import from a large package like `@mui/icons-material` can cause **the entire module** to be parsed and analyzed—even if you only needed one icon. This leads to an explosion of open file handles, especially when Vitest is running in parallel across many threads.

### Problematic (Named) Import – ❌

```ts
// This opens the entire icon module behind the scenes
import { Delete } from '@mui/icons-material';
```

### Correct (Default) Import – ✅

```ts
// This only opens the specific icon file
import Delete from '@mui/icons-material/Delete';
```

The same applies for other large packages, such as components from `@mui/material`.

### More Invalid vs Valid Examples

```ts
// ❌ Named import (bad)
import { Button } from '@mui/material';

// ✅ Default import (good)
import Button from '@mui/material/Button';
```

## Why This Happens

When Vitest runs your tests, it analyzes and processes all the imported files. With named imports, tools like Vite (and by extension Vitest) may resolve the full package entry point and end up opening hundreds or even thousands of files—especially for large packages like MUI. Multiply that across many test files and parallel threads, and you can quickly hit your OS file descriptor limit.

## The Fix

1. **Replace all named imports** from MUI packages with default imports from the specific component or icon path.
2. Optionally, increase your OS file descriptor limit (ulimit), though this only postpones the underlying problem.
3. Restart your test runner and watch the errors disappear.
