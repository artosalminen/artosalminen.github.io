---
title: "How to resolve Intel Graphics Command Center port conflict"
date: 2024-01-10
series: ["Coding", "Windows"]
weight: 1
tags: ["Coding", "Windows"]
author: "Arto Salminen"
---


For developers working on .NET Core and other development servers, encountering port conflicts can be a common issue. Recently, I came accross with the default port (5000) used for local development server and the Intel(R) Graphics Command Center server. In this blog post, I'll introduce a workaround that involves editing the Windows registry to resolve the conflict and ensure smooth development processes.

**Understanding the Issue:**

The Intel(R) Graphics Command Center server, specifically the OneApp.IGCC.WinService.exe executable, by default, utilizes port 5000. However, this port is commonly used for development servers in the .NET Core ecosystem and other software development environments. When both the Intel(R) Graphics Command Center server and a development server attempt to bind to port 5000 simultaneously, a conflict arises.

**The Workaround:**

To address this issue, we can implement a workaround by editing the Windows registry and appending the --urls parameter to the startup command of the Intel(R) Graphics Command Center service executable. This involves specifying an alternative port, such as 50000, to avoid clashes with the default port used for development servers.

Here are the step-by-step instructions:

1. **Accessing the Windows Registry:**
   - Press `Win + R` to open the Run dialog.
   - Type `regedit` and press Enter to open the Registry Editor.
   - Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OneApp.IGCC.WinService`.

2. **Editing the Registry Entry:**
   - Locate the `ImagePath` entry on the right-hand side, which holds the path to the OneApp.IGCC.WinService.exe executable.
   - Append `--urls http://127.0.0.1:50000` to the existing path. Ensure that there is a space before the double dash (--).

   Example:
   ```
   "C:\Path\To\OneApp.IGCC.WinService.exe" --urls http://127.0.0.1:50000
   ```

   Save the changes.

3. **Restart the Service:**
   - Restart the Intel(R) Graphics Command Center service to apply the new configuration.
   - Open the Run dialog again (`Win + R`), type `services.msc`, and press Enter.
   - Locate the Intel(R) Graphics Command Center service, right-click, and choose Restart.

By following these steps, you have successfully configured the Intel(R) Graphics Command Center server to use an alternative port (50000), avoiding conflicts with the default port used for .NET Core and other development servers.

**Conclusion:**

Port conflicts can be a hurdle for developers working on various projects simultaneously. This workaround provides a solution for users encountering conflicts between the Intel(R) Graphics Command Center server and development servers using the default port 5000. By making a simple registry edit, you can ensure a smoother development experience without compromising the functionality of either service.

