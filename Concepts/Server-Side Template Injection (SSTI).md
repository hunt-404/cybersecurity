___

# Level 1 - Basics
For example:
> Hello, `{{name}}`!

If the user enters `Ali`, the server replaces `{{name}}` with `Ali`:
> Hello, Ali!

### So what is SSTI?
**SSTI happens when the server accidentally treats the user's input as template instructions instead of just text.**
Imagine a website asks:
> Enter your name:

You enter:
```text
{{7*7}}
```

If the website safely treats it as text, you would see:
> Hello, {{7*7}}!

But if the website passes it to the template engine incorrectly, the engine may interpret it as a command and produce:
> Hello, 49!

That's **Server-Side Template Injection**.
#### Simple analogy
Imagine you're filling out a form:
- **Safe:** You give the worker a note saying `{{7*7}}`, and they print those exact characters.
- **SSTI:** The worker mistakes your note for **instructions** and performs `7 × 7`.

So, in one sentence:

> **SSTI is when a website lets user input become template code that the server executes/interprets, instead of treating it as ordinary text.**

> [!danger] Danger
> It can be dangerous because, depending on the template engine and how the application is configured, an attacker may be able to make the server perform unintended actions.

### What Changes With SSTI?
Suppose the application constructs a template using attacker-controlled input.
Conceptually:
```text
Attacker input
      ↓
Application inserts it into template
      ↓
Template engine parses it
      ↓
Attacker-controlled expression executes
```

The crucial security boundary is therefore:
```text
          SAFE
           ↓
Template → Data → Rendering


          DANGEROUS
           ↓
User input → Template → Rendering
```
That's the fundamental concept you need to remember.

### What is a template engine?

A **template engine** processes templates of code and combines them with application data.
Common examples include:

| Technology | Examples             |
| ---------- | -------------------- |
| Python     | Jinja2, Mako         |
| Java       | Freemarker, Velocity |
| PHP        | Twig, Smarty         |
| Ruby       | ERB                  |
| JavaScript | Handlebars, Pug      |
The exact syntax and capabilities depend on the engine.

### Template Data vs Template Code
| Aspect               | Template Data                                  | Template Code                                         |
| -------------------- | ---------------------------------------------- | ----------------------------------------------------- |
| **What it is**       | Information that the template displays or uses | Instructions that tell the template engine what to do |
| **Purpose**          | Provides values/content                        | Controls how the template works                       |
| **Example**          | `Ali`                                          | `{{name}}`                                            |
| **Another example**  | `user.is_admin`                                | `{% if user.is_admin %}`                              |
| **How it's treated** | Treated as ordinary data                       | Interpreted by the template engine                    |
| **Example input**    | `{{7*7}}` as plain text                        | `{{7*7}}` interpreted as a template expression        |
| **Expected result**  | Displays `{{7*7}}`                             | May produce `49`                                      |
| **SSTI relevance**   | Safe when kept as data                         | Dangerous when untrusted user input becomes code      |

> [!IMPORTANT]
> **Template Data** = "What to display."  
> **Template Code** = "What to do."  
> **SSTI** = When user-controlled data is accidentally treated as template code.


## Security Perspective
SSTI matters because template engines may provide capabilities far beyond simple text substitution.
Depending on:
- Template engine
- Configuration
- Application design
- Available objects/functions
- Sandbox restrictions

template expressions may potentially access:
```text
Variables
   ↓
Objects
   ↓
Application functionality
   ↓
Sensitive data
   ↓
Server-side capabilities
```

In severe cases, SSTI can become **server-side code execution**.

But don't make this mistake:
> **SSTI ≠ automatically RCE.**

A professional pentester asks:
```text
Is template evaluation happening?
          ↓
Which engine?
          ↓
What can the expression access?
          ↓
Are dangerous capabilities exposed?
          ↓
What is the actual impact?
```

### Practical Relevance
When you eventually encounter a web application during an OSCP-style assessment, don't immediately think:
> "What SSTI payload should I try?"

Think:
> **"Where does user input enter the rendering pipeline?"**

Potential areas worth noticing include:
- Username/profile rendering
- Custom error pages
- Email templates
- Report generation
- Preview functionality
- Notification systems
- CMS features
- Admin-configurable templates
- Search/result formatting
- PDF/document generation
These features often involve server-side rendering.

### Key Takeaways
Remember these **five ideas**:
1. **Templates generate dynamic content.**
2. **Template engines interpret template syntax.**
3. **SSTI occurs when attacker-controlled input becomes template code.**
4. **SSTI impact depends heavily on the engine and its exposed capabilities.**
5. **A pentester should identify the rendering boundary before attempting exploitation.**

Your foundational mental model:
```text
SSTI question:

"Can my DATA cross the boundary
and become TEMPLATE CODE?"
```

# Level 2 —  Internal Architecture

> **SSTI happens when attacker-controlled data crosses into the template-code boundary.**

Now we need to understand **where that boundary exists and what happens internally**.
A useful model is:
```text
HTTP Request
     │
     ▼
┌──────────────┐
│ Web Framework│
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Application  │
│    Logic     │
└──────┬───────┘
       │
       ├───────────────┐
       │               │
       ▼               ▼
 Template           User Data
       │               │
       └───────┬───────┘
               ▼
       ┌──────────────┐
       │   Template   │
       │    Engine    │
       └──────┬───────┘
              │
              ▼
       Rendered Response
```

The important question is:
**Does the application give the template engine attacker-controlled _data_, or attacker-controlled _template source_?**

