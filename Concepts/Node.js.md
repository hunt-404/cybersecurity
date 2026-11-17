___
# Node.js — Level 1: Foundations
**Node.js is a runtime environment that allows developers to run JavaScript outside the web browser.**

Normally:
```
JavaScript
      |
      v
Browser
(Chrome, Firefox)
```

JavaScript was originally created to run inside browsers.

Node.js changed this:
```
JavaScript
      |
      v
Node.js Runtime
      |
      v
Server / Backend Application
```

Now developers can use JavaScript to build:
- Web servers
- APIs
- Real-time applications
- Automation tools
- Command-line applications

### Core Concepts
| Section            | Explanation                                                                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Simple Meaning     | Node.js lets JavaScript run on a server instead of only inside a browser.                                                                       |
| Purpose            | Build fast, scalable backend applications using JavaScript.                                                                                     |
| Mental Model       | A JavaScript engine + tools that allow JavaScript to communicate with the operating system.                                                     |
| Technical View     | A runtime based on Google's V8 JavaScript engine with an event-driven, non-blocking I/O architecture.                                           |
| Security Relevance | Node.js applications often expose APIs, process user input, handle files, and use many third-party packages, creating possible attack surfaces. |
### Why Was Node.js Created?
Before Node.js:
A common web architecture:
```
User
 |
 v
Browser
 |
 v
Web Server
 |
 v
Backend Language
(PHP, Java, Python)
 |
 v
Database
```

JavaScript was only for frontend:
```
Browser
 |
 └── JavaScript
      (buttons, animations, validation)
```

Developers wanted:
- JavaScript on both frontend and backend
- Faster development
- Real-time applications
- Efficient handling of many connections

Node.js provided this:
```
Frontend
JavaScript
    |
    |
Backend
Node.js
    |
    |
Database
```
One language across the entire application.

### Components of Node.js
| Component                      | What It Is                                                                   | Main Function / Examples                                                                                                       | Security Relevance                                                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| **V8 JavaScript Engine**       | Google's JavaScript engine, used by Chrome and Node.js                       | Converts JavaScript code into machine instructions that the computer can execute                                               | Understanding V8 helps explain how Node.js executes JavaScript                                                     |
| **Event Loop**                 | The mechanism that allows Node.js to handle many operations without blocking | Manages asynchronous tasks such as database queries, file operations, and API requests; executes callbacks when tasks complete | Blocking operations can slow down or affect the responsiveness of a Node.js application                            |
| **Node APIs**                  | Built-in APIs that allow Node.js applications to interact with the system    | File system, networking, operating system, processes, environment variables, and cryptography                                  | APIs that interact with the OS require careful input and access control                                            |
| **npm (Node Package Manager)** | Node.js's package manager and package ecosystem                              | Installs and manages libraries such as Express, database drivers, and authentication packages                                  | Dependencies can contain vulnerabilities; package versions and dependencies are important during security analysis |
| **Modules**                    | Reusable pieces of JavaScript functionality used by Node.js applications     | Applications can be split into built-in, local, and third-party modules                                                        | Understanding modules helps identify application functionality and external dependencies                           |

