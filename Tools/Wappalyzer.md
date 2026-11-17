___
# Wappalyzer — Level 1: Foundations
**Wappalyzer is a technology detection tool that identifies the technologies used by a website.**

In simple words:
> Wappalyzer helps you answer: **"What is this website built with?"**

A website looks simple from the outside, but behind it there may be:
```
Website
   |
   ├── Web Server (Apache, Nginx)
   ├── Programming Language (PHP, Python, JavaScript)
   ├── Framework (Laravel, Django, React)
   ├── CMS (WordPress, Joomla)
   ├── Database clues
   ├── Analytics tools
   ├── Security tools
   └── Hosting/CDN information
```

Wappalyzer tries to discover these technologies by analyzing the website.
### What is it?
**Wappalyzer = Web technology profiler**

It is a tool used to detect:
- Content Management Systems (CMS)
- Web frameworks
- JavaScript libraries
- Programming languages
- Web servers
- E-commerce platforms
- Analytics platforms
- Security technologies

### Why does it exist?
Websites expose small technical clues.

For example:
A website response may contain:
```bash
<script src="/wp-content/themes/theme.js">
```

Wappalyzer recognizes:
```
wp-content
      ↓
WordPress
```

Or a server response might show:
```
Server: nginx
```

Wappalyzer identifies:
```
Web Server:
nginx
```
The internet is full of these tiny fingerprints. Wappalyzer simply collects and interprets them, because apparently even websites leave little breadcrumbs everywhere.

### How Does Wappalyzer Work?
A simplified process:
```
You enter website URL
          |
          v
Wappalyzer scans website
          |
          v
Collects fingerprints
          |
          v
Matches against technology database
          |
          v
Shows detected technologies
```

Example:
Target:
```
https://example.com
```

Wappalyzer result:
```
CMS:
WordPress

Web Framework:
Laravel

JavaScript Library:
React

Web Server:
Nginx

Analytics:
Google Analytics
```

## Security Perspective (OSCP Mindset)
For penetration testing, Wappalyzer is an **information gathering tool**.
It belongs in the:
```
Reconnaissance
       ↓
Enumeration
       ↓
Vulnerability Analysis
```
phase.

You are not exploiting anything yet.
You are building a map.