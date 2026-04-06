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

You have found the **"Injector"**! This is the exact moment the "Pipe" is connected to the "Main Line" (the browser's document head). 

This is a common pattern in libraries like jQuery or Zabbix's internal loaders: they create a `<script>` tag, attach it to the page to make it execute, and then immediately remove the tag to keep the HTML "clean."

To determine if this is a **Pass** or a **Fail** for **Client-Side Resource Manipulation** or **JS Execution**, we need to find out what was inside that `script` variable right before it was appended.

---

### 1. The Trace Strategy: Finding the "Script Content"
The `script` variable was created and configured a few lines *above* this one. 

1.  **Look Upward:** Scroll up from the `appendChild` line.
2.  **Search for:** `script.src =` or `script.text =`.
3.  **The Goal:** We need to see if the source of the script is a hardcoded file or something from the URL.

---

### 2. Identifying the Intake (The Source)
As you look at the lines above, apply these **Stop Rules**:

#### ✅ Scenario A: THE PASS (Secure)
If you see:
```javascript
script.src = "js/common.js"; 
// OR
script.text = "console.log('init');";
```
* **Why:** The content is hardcoded in the script itself. No attacker can "pour" their own code into this pipe.
* **Verdict:** **STOP.** This is a **PASS**.



#### ❌ Scenario B: THE FAIL (Vulnerable)
If you see:
```javascript
script.src = (new URLSearchParams(window.location.search)).get("include");
```
* **Why:** The "Intake" is pulling a filename directly from the URL. An attacker could use `?include=https://evil.com/malware.js`.
* **Verdict:** **FAIL.** This is **Client-Side Resource Manipulation**.

---

### 3. The "Live" Trace (The Call Stack)
If the code above the line is confusing (e.g., it just says `script.src = url;`), use the **Call Stack**:

1.  **Set a Breakpoint** on the `doc.head.appendChild( script )` line.
2.  **Trigger the action** (usually a page refresh).
3.  When it pauses, hover your mouse over the word `script`.
4.  **Check the `src` property:** Does it point to a local file you recognize, or an external URL?
5.  **Check the `text` property:** Does it contain code that looks like it has your URL parameters inside it?



---

### 4. Summary Checklist for this Sink
* **The Tap:** `doc.head.appendChild( script )`
* **The Trace:** Find where `script.src` or `script.text` is set.
* **The Result:**
    * **PASS:** If the script loads a local, hardcoded file.
    * **FAIL:** If the script loads a file or executes code based on `location.search` or `location.hash`.

**What are the 5 lines of code directly above that `appendChild` line?** Copy and paste them here, and we can finish this trace together.

You have found a **JavaScript Execution Sink**. In the plumber's analogy, this is the "Electric Motor" of the pipe system. Instead of just moving water (data), this line takes the "Water" (`code`) and turns it into **Action** (running a script).

This is a critical moment in your audit. If an attacker can get their own "Water" into that `code` variable, they have achieved **DOM-based XSS**.

---

### 1. The "Water System" Analysis
* **The Intake (Source):** Where the `code` variable is first defined.
* **The Pipe (The Flow):** Any functions that process the `code` string.
* **The Tap (The Sink):** `script.text = code;` (followed by the `appendChild` you found earlier).



---

### 2. Trace the "Intake" (The Source)
You need to find out where that `code` string comes from. Look at the lines directly above `script.text = code;`.

**Search Strategy (`Ctrl + Shift + F`):**
Search for: `code =` or `var code =`.

#### ✅ Scenario A: THE PASS (Secure)
If you see:
```javascript
var code = "console.log('Zabbix Initialized');";
```
* **Why:** The code is a hardcoded string. No matter what the user puts in the URL, this string never changes.
* **Verdict:** **STOP.** This is a **PASS**.

#### ❌ Scenario B: THE FAIL (Vulnerable)
If you see:
```javascript
var code = (new URLSearchParams(window.location.search)).get("cb");
```
* **Why:** The "Intake" pulls directly from a URL parameter. An attacker can visit `site.com/?cb=alert(document.domain)` and it will execute.
* **Verdict:** **FAIL.**

---

### 3. The "Live" Trace (The Call Stack)
If the code is complex (e.g., `code` is passed into a function as an argument), use the **Call Stack**:

1.  **Set a Breakpoint** on the line: `script.text = code;`.
2.  **Trigger the action** (usually a page refresh).
3.  When it pauses, look at the **Scope** pane. What is the value of `code`?
4.  **Click the function below** in the **Call Stack**.
5.  Repeat until you find the very first place the `code` string was created.



---

### 4. Special Case: AJAX/Server-Side Data
In many professional apps like Zabbix, `code` comes from an **AJAX response** (the server sends a piece of JS to run).

* **The Trace:** You might see `code = response.data;`.
* **The Verdict:** If the server is sending the code, this is technically a **PASS** for *Client-Side* security. You've proven the client isn't the one creating the vulnerability; the "Water" is coming clean from the "City Main" (the Server).

---

### Summary Checklist for this Trace
* **Sink:** `script.text` (The Motor).
* **Trace:** Use the **Call Stack** to find the origin of the `code` variable.
* **Verdict:**
    * **PASS:** If the code is hardcoded or comes from a trusted Server Response.
    * **FAIL:** If any part of the `code` string contains data from `location.hash` or `location.search`.

**What does the line immediately above `script.text = code;` say?** Does it show where `code` was defined? Copy that line here and we can finish this "Clean" audit item.
