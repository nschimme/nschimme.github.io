---
layout: post
date: 2026-05-08
title: Bringing Secure Password Storage to WebAssembly with Qt6
tags:
  - qt6
  - opensource
  - hacker
---
One of the greatest promises of modern development is the "write once, run anywhere" philosophy. With Qt6 and WebAssembly (Wasm), we are closer than ever to making high-performance desktop applications run seamlessly in a web browser. But as any developer who has ported a C++ app to the web knows, there’s always a "but." 

For many, that "but" was **secure password storage.**

On Windows, we have the Credential Manager. On macOS, the Keychain. On Linux, libsecret or KWallet. But when your app runs in a browser sandbox, those system APIs are out of reach. Until now, WebAssembly versions of Qt apps often had to compromise on security or skip persistent logins entirely. 

I’m excited to share that I’ve been working on a solution to bridge this gap. My recent contribution to **QtKeychain** adds native WebAssembly support, ensuring that "cross-platform" truly means *every* platform.

### See it in Action

Before we dive into the "how," you can see the results for yourself. I've hosted a live demo of a Qt6 application running in the browser that utilizes this new backend to securely store and retrieve credentials.

1. **Click Demo URL:** [https://docs.mume.org/MMapper/beta/](https://docs.mume.org/MMapper/beta/)
2. **Navigate to Settings:** Go to **Edit > Preferences** in the top menu bar.
3. **Open Account Manager:** Click the **Manage Account** button.
4. **Trigger the Bridge:** Enter an account name (e.g., "Tester") and click **OK**.
5. **Credential Entry:** - A transient HTML modal will appear. This is the **Bridge UI**.
   - Enter a password and click **Save**.
6. **Native Storage:** Observe the browser's native "Save Password" prompt (Chrome/Firefox/Safari). Confirm the save to store the secret in your browser's secure vault.
7. **Persistence Check:** Agree to the prompt for **Automatic Logins**.
8. **Verify Retrieval:** Refresh the page. The application will now automatically retrieve your credentials via the `navigator.credentials` API, bypassing the need for manual entry.

### The Challenge: Security in a Sandbox

The browser is a hostile environment by design. Applications cannot simply reach out and touch the local file system or call OS-level encryption APIs. To store a password securely in a way that integrates with the user's life, we have to play by the browser's rules.

The goal was simple: when a user saves a password in a Wasm-based Qt app, it should feel native. It should trigger the browser’s own "Save Password" prompt and work with existing password managers like 1Password or Bitwarden.

### The Solution: The HTML Bridge

To make this work, I implemented a bridge between the Qt/C++ world and the browser's capabilities. Since WebAssembly operates within the JavaScript context of the page, we can leverage the browser's native storage mechanisms.

Here is how the flow works:

1.  **The Request:** When your Qt code calls `WritePasswordJob`, the Wasm backend of QtKeychain doesn't try to write to a hidden file.
2.  **The Bridge:** It interacts with the browser's credential storage (such as IndexedDB or the Credential Management API) via emscripten-based calls.
3.  **The User Experience:** The browser handles the sensitive data, and for the user, it "just works"—credentials can be persisted across sessions without the developer needing to roll their own (often insecure) encryption logic.
4.  **The Retrieval:** When the app restarts, `ReadPasswordJob` queries the store, and the password is fed back into your C++ logic seamlessly.

### Why This Matters for Developers

If you are building a cross-platform app today, security shouldn't be an afterthought or a platform-specific headache. By integrating this into QtKeychain, we’ve made secure storage a "just works" feature. 

* **No Conditional Compiling:** You don't need `#ifdef Q_OS_WASM` all over your login logic. The same QtKeychain API you use for your Windows or macOS builds now handles the web automatically.
* **User Trust:** Users are rightfully wary of entering passwords into web apps. By using the native browser-supported storage pathways, you leverage the security model the user already trusts.
* **True Portability:** This removes one of the final blockers for migrating complex, data-driven desktop applications to the web.

### Looking Forward

This update is part of a broader effort to ensure that the Qt ecosystem remains the best choice for developers who value both performance and reach. Secure password storage is a fundamental building block, and I'm thrilled to contribute this piece to the puzzle.

Whether you're building a specialized tool or a community-driven project, the barrier to "secure by default" just got a lot lower.

*Check out the full implementation and join the discussion over on [GitHub Pull Request #295](https://github.com/frankosterfeld/qtkeychain/pull/295).*