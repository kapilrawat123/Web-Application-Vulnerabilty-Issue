# Introduction

Server-side template Injection, or SSTI, is a vulnerability that occurs when user input is injected into a template engine of an application. This can lead to a range of security issues, including code execution, data exposure, privilege escalation, and Denial of Service (DoS). SSTI vulnerabilities are often found in web applications that use template engines to generate dynamic content and can have serious consequences if left unaddressed.

Server-Side Template Injection (SSTI) vulnerability occurs when user input is unsafely incorporated into a server-side template, allowing attackers to inject and execute arbitrary code on the server. Template engines are commonly used in web applications to generate dynamic HTML by combining fixed templates with dynamic data. When these engines process user input without proper sanitization, they become susceptible to SSTI attacks.


# **Core Concepts of SSTI**

- **Dynamic Content Generation:** Template engines replace placeholders with actual data, allowing applications to generate dynamic HTML pages. This process can be exploited if user inputs are not properly sanitized.
- **User Input as Template Code:** When user inputs are treated as part of the template code, they can introduce harmful logic into the rendered output, leading to SSTI.

**Core Concepts of SSTI**

- **Dynamic Content Generation:** Template engines replace placeholders with actual data, allowing applications to generate dynamic HTML pages. This process can be exploited if user inputs are not properly sanitized.
- **User Input as Template Code:** When user inputs are treated as part of the template code, they can introduce harmful logic into the rendered output, leading to SSTI.

The core of SSTI lies in the improper handling of user input within server-side templates. Template engines interpret and execute embedded expressions to generate dynamic content. If an attacker can inject malicious payloads into these expressions, they can manipulate the server-side logic and potentially execute arbitrary code.

---
### Flow of an SSTI Attack

![Flow of an SSTI attack](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1717984475799)  

When user input is directly embedded in templates without proper validation or escaping, attackers can craft payloads that alter the template's behaviour. This can lead to various unintended server-side actions, including:

- Reading or modifying server-side files.
- Executing system commands.
- Accessing sensitive information (e.g., environment variables, database credentials).
---

# **Template Engines**

A template engine is like a machine that helps build web pages dynamically. Here's how it works in simple terms:

Imagine you're making a birthday card for a friend. You want to include their name, age, and a personalized message. Instead of writing a new card from scratch, you use a template with placeholders for the name, age, and message.

A template engine works similarly:

1. **Template**: The engine uses a pre-designed template with placeholders like {{ name }} for dynamic content.
2. **User Input**: The engine receives user input (like a name, age, or message) and stores it in a variable.
3. **Combination**: The engine combines the template with the user input, replacing the placeholders with the actual data.
4. **Output**: The engine generates a final, dynamic web page with the user's input inserted into the template.

![Template engine analogy](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1717984573585)

Template engines offer various functionalities that speed up the development process but can also introduce risks.

> [!NOTE]
> Most template engines allow expressions to be used for simple calculations or logic operations within templates.


In the context of SSTI, the template engine's ability to execute code is what makes it vulnerable to attacks. If user input is not properly sanitized, an attacker can inject malicious code, which the template engine will execute, leading to unintended consequences.

### Common Template Engines

Template engines are an integral part of modern web development, allowing developers to generate dynamic HTML content by combining templates with data. Here are some of the most commonly used template engines:

- **Jinja2**: Highly popular in Python applications, known for its expressiveness and powerful rendering capabilities.
- **Twig**: The default template engine for Symfony in PHP, Twig offers a robust environment with secure default settings.
- **Pug/Jade**: Known for its minimal and clean HTML templating syntax, Pug/Jade is popular among Node.js developers.


# How Template Engines Parse and Process Inputs

Template engines work by parsing template files, which contain static content mixed with special syntax for dynamic content. When rendering a template, the engine replaces the dynamic parts with actual data provided at runtime. For example:

```python
from jinja2 import Template

hello_template = Template("Hello, {{ name }}!")
output = hello_template.render(name="World")
print(output)
```

In this example, `{{ name }}` is a placeholder that gets replaced with the value `"World"` during rendering.

# Determining the Template Engine

Different template engines have distinct syntaxes and features, making them vulnerable to SSTI in various ways. Here are some examples of vulnerable template syntaxes:

**Jinja2/Twig**

Jinja2 and Twig are similar in syntax and behavior, making them somewhat challenging to distinguish from each other just by payload responses. However, you can detect their presence by testing their expression-handling capabilities. For example, using the vulnerable VM, if you use the payload {{7*'7'}} in Twig, the output would be 49.

![Output of the payload is 49](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1716692887811)

However, if you use the same payload in an application that uses Jinja2, the output would be 7777777

![Output of the payload is 7777777](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1716692888281)  

**Jade/Pug**

Pug, formerly known as Jade, uses a different syntax for handling expressions, which can be exploited to identify its usage. Pug/Jade evaluates JavaScript expressions within `#{}`. For example, using the payload #{7*7} would return 49.

![Output of the payload is 49](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1716692888719)  

Unlike Jinja2 or Twig, Pug/Jade directly allows JavaScript execution within its templates without the need for additional delimiters like {{ }}. For example:

![Direct command execution](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1716692889151)



# **Technique for  Identify Template**

In most cases, this polyglot payload will trigger an error in presence of a SSTI vulnerability:

```
${{<%[%'"}}%\.
```


Paste that error in Chat GPT it will tells you which type of Template that website use.
