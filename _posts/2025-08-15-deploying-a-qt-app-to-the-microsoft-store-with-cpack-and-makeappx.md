---
layout: post
date: 2025-08-15
title: Deploying a Qt App to the Microsoft Store with CPack and MakeAppx
tags:
  - qt
  - appx
  - msix
  - cmake
  - cpack
---
As the maintainer of an open source Qt application, I’ve run into a common problem: Windows users often get blocked by security warnings because the software isn’t digitally signed. The cost of a code signing certificate can be prohibitive, but there’s a fantastic alternative — the Microsoft Store. By submitting your app to the Store, Microsoft signs it for you, eliminating trust warnings and giving users a clean, familiar installation experience.

This guide explains how I replaced my Windows NSIS installer with a Microsoft Store–ready package. The Store doesn’t want a classic installer — it wants an `.appxupload` file. Fortunately, Qt already works with CMake and CPack, so we can swap out the CPack generator and use `MakeAppx.exe` to produce a Store package without rewriting the build system.

Here’s the workflow I used to go from an NSIS-based `.exe` to a Store-ready `.appxupload` with minimal effort.

## **1\. Prerequisites**

You’ll need:

\- A working Qt + CMake build on Windows

\- Visual Studio (for `MakeAppx.exe`)

\- A Microsoft Partner Center account for Store submission

## **2\. Adjusting** `CMakeLists.txt` **for CPack**

If you were using NSIS before, you might have had something like:

\`\`\`cmake

set(CPACK\_GENERATOR "NSIS")

\`\`\`

We're going to replace it with:

\`\`\`cmake

set(CPACK\_GENERATOR "External")

\`\`\`

Why **External**? Because the Microsoft Store packaging process is not handled by a built-in CPack generator. Instead, we instruct CPack to run a custom script that orchestrates the entire packaging workflow. This script finds all the necessary files and uses the `MakeAppx.exe` utility to build the final package.

Reference this commit to see all of the necessary changes to invoke the `MakeAppX.cmake` external script: [https://github.com/MUME/MMapper/commit/a86804cc6c1f57eae272a181d4e474574ea248d4](https://github.com/MUME/MMapper/commit/a86804cc6c1f57eae272a181d4e474574ea248d4)

## 3\. Submitting to the Microsoft Store

Once you have your `.appxupload`:

1\. Go to Partner Center

2\. Create a new app or update an existing one

3\. Upload your `.appxupload` file

4\. Fill in your Store listing and submit for certification

Microsoft typically takes about a week to approve a new submission. Once approved, you can follow the same process for updates, and they’ll appear automatically for all users.

## 4\. Why This Approach Wins

This process solves two of the biggest headaches for open source Windows projects:

1.  **No expensive code signing certificate** — The Store signs the app for you, so users never see a “publisher unknown” warning.
    
2.  **Automatic updates** — The Store handles seamless upgrades, keeping all users on the latest version without any manual downloads or installers.
    

Once I understood the flow, the transition from NSIS to a successful Store submission took less than a week. The result: a trustworthy install process for users, less maintenance for me, and a deployment pipeline that’s future proof-- if still a bit manual on the submission portion.