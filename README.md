# Client-Side-Resource-Manipulation

If you aren't finding `.src =`, it’s often because modern applications (like Zabbix) use more abstract methods to load resources. They might use "Loaders," "Appenders," or even simple HTML tags that are already on the page.

Here is the "Clean" end-to-end way to test for **Client-Side Resource Manipulation** when the simple search fails.

---

### 1. The Alternative Search (The "Pipe" Search)
If the code doesn't explicitly type `.src =`, it might be using these methods to "inject" a new script or iframe into the page. Search for these instead:

* **`createElement('script')`**: This is how the "Pipe" is built before the source is even added.
* **`createElement('iframe')`**: Same for frames.
* **`appendChild(`**: This is the moment the "Pipe" is connected to the "Main Line" (the DOM).

**The Trace:** Once you find `const s = document.createElement('script')`, look at the next few lines. You will eventually see `s.src = someVariable;`.

---

### 2. The Network Trace (The "Observation" Method)
Instead of searching for the code, look at what the "Water" is actually doing in the **Network** tab.

1.  Open **Network** tab in DevTools.
2.  Filter by **JS** or **Doc** (for iframes).
3.  Refresh the page.
4.  **Look for "Dynamic" URLs:** Do any of the scripts being loaded have a version number or a language code in the URL? (e.g., `.../jsloader.php?lang=en&v=7.0.1`).
5.  **The Test:** Go to your browser's address bar and add `?lang=../../evil.com/test`.
6.  **Verdict:**
    * **FAIL:** If the **Network** tab shows a new request trying to hit `evil.com`.
    * **PASS:** If the browser still loads `lang=en` or gives a `404` error because it only accepts specific strings.



---

### 3. Testing for "Cross-Site Flashing" (The "Legacy" Resource)
This is on your list but rarely uses `.src =` anymore. It usually involves **Adobe Flash** or **Silverlight** (which are mostly dead), but in modern apps, it applies to **PDF Embeds**.

* **Search for:** `<embed`, `<object`, or `PDFJS`.
* **The Test:** If you see a PDF viewer, check if the URL looks like `viewer.html?file=report.pdf`.
* **The Attack:** Change it to `viewer.html?file=https://evil.com/malicious.pdf`.
* **Verdict:** If the viewer tries to load the external PDF, it’s a **FAIL**.

---

### 4. Summary Checklist for Resource Manipulation

| Method | What to look for | Result |
| :--- | :--- | :--- |
| **Code Search** | `createElement('script')` | **FAIL** if `.src` is set using a URL parameter. |
| **Network Tab** | Requests with `lang=` or `file=` | **FAIL** if you can change the domain of the request. |
| **DOM Tree** | `<iframe>` tags in the **Elements** tab | **FAIL** if the `src` attribute changes based on your URL input. |

---

### Where to Stop?
If you find that all scripts and resources are loaded from the **Same Origin** (your own domain) and the filenames are **Hardcoded** (like `common.js`, `main.js`), you can **STOP**. This is a **PASS**.

**Try searching for `createElement` now.** If you find a result, paste the 3 lines of code below it—that’s where the "Intake" will be hiding!
