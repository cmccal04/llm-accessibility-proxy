# Networks Final Project  
## LLM Powered Accessibility Repair HTTPS Proxy  

**Contributors:** Cullen McCaleb, Luke Brennan, Vivian Lau

---
## Note
This repository only contains this README file in order explain this project in more detail than my resume can. Unfortunately, this code cannot be shared publicly as the Tufts University LLMProxy library being used must be kept private. Additionally, the proxy.c code cannot be shared so as not to reveal a solution to future students in the Tufts Networks course. Please email me at cullenmcc121@gmail.com for any questions about this project!

---

## Overview
Our proxy leverages LLM functionality to improve the accessibility of web pages it retrieves. When the client requests a webpage, the proxy will inject JavaScript code to add a button to the page. This button can be focused on using keyboard navigation: accessible to all users but not automatically visible and thus would not impact regular usability of the website to those who are not visually impaired. 

As soon as the webpage loads, the injected JavaScript will:

1. Download Deque System's Axe-core accessibility tool `axe.min.js` (which reads a website for its accessibility issues)
2. Run `axe.min.js` on the HTML to produce a list of issues in JSON format. These suggested issues include some context around what accessibility rule is violated, as well as the HTML snippet containing the violation.
3. Send the accessibility issues to a Python server running parallel to our C proxy.

Python sends the issues to LLMProxy. The context is a combination of RAG context provided by the Axe-core API, and documentation on the layout of the input we give the LLM. This input, the query, is simply the aforementioned JSON list of accessibility violations. We also provide a system prompt to specify the format of the answer for the LLM, which is an array in JSON format containing two values: a `dom_selector` and the fixed HTML code. The `dom_selector` specifies where the patch should be applied.

If the "Run Accessibility Repair" button is pressed, the JavaScript will poll the Python server until the result from the LLM is ready. The suggested patches are sent back to the JavaScript, which applies them to the web page.


The LLM performs well on HTML snippets which themselves contain, or have parent HTML elements which contain, all necessary context to make a fix. For example, adding button labels. It does not work well on fixes involving different parts of the HTML body, such as color contrast of some element against the entire page background.

---

## Dependencies
- `flask`  
- Tufts University's `LLMProxy`

---

## Usage
1. Compile the code using:
```
   make
```
2. In one terminal window, run:
```
   ./proxy <port> <ca_cert_path> <ca_key_path>
```
3. In another terminal window, run:
```
   python3 a11y_server.py
```
**Any time you want to try the proxy on a page you must kill the C proxy in terminal and restart it, then reload the web page.** 

If you want to benchmark the proxy yourself (the following are *not* required dependencies for the proxy to work):
1. Download the axe DevTools extension on FireFox.
2. Inspect the web page you are visiting and run axe DevTools scan.
3. Tabbing to the button is most efficient if you first click in the top left corner, then hit Tab, then hit Shift+Tab until the button appears in the top left.
4. Press enter to run the repair.
5. When the repair is done, re-run axe DevTools scan and see the improvement!
---

## Files
- **a11y_server.py** - Communicates with the proxy to receive a payload and format it properly before sending it to the LLM.
- **fixer.js** - All necessary JavaScript code to update the HTML of the fetched webpage.
- **llmrunner.py** - Offers `fix_prompt()` function to provide context for the LLM and prompt it to produce the necessary HTML fixes. Used by `a11y_server.py`.
- **Makefile** - Makefile to compile the code.
- **proxy.c** - Implements the HTTPS proxy in C. Uses `fixer.js` and `a11y_server.py`.
