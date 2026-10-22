# THM capstone Naham Store

Capstone for Cybersecurity path, I do this as beginner so there's a lot of googling.

### Task 1: Recon

Finding: Jimmy Jones SSN

**Listing Open Port / System Version:**

Using `nmap -sV -sC -p- -oA scan <target_ip>`

Found open port
22/tcp, 80/tcp, 8000/tcp
System: nginx/1.18.0 (Ubuntu)

I know now the system so I tried to google any known vuln for the `nginx/1.18.0` version

**Listing Hidden Directory**

Using ffuf

I run `ffuf -u <target_domain>/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -ac` , to have Directory discovery and found the following:
- basket
- css
- js
- login
- logout
- register
- robots.txt
- returns
- search
- staff
- uploads

I run `ffuf -u <target_domain>:8000/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -ac` , to have Directory discovery and found the following:
- admin 
- robots.txt


So i check `<target_domain>:8000/admin/` and it gives me login form, i try common password and username as follow:
- `admin:admin` (this makes me login)
- `admin:password`
- `admin:admin123`
- `admin:blank`


I run `ffuf -u <target_domain> -H "Host: FUZZ.<target_domain>" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -ac`, to get list of subdomains and found the following:
- .htaccessYfWmZWsr
- WWW
- stock (Api endpoint)
- marketing


### Task 2: XSS

I navigate manually the following subdomains
- stock (nothing usual already test any know vuln)
- marketing (found list of campaigns)

Interesting finding: **marketing.nahamstore.thm**
- it have inputs for name and email address, tried adding some payloads, with simple js alert, but nothing respond

- tried to change the ending letter in the url:
    - `http://marketing.nahamstore.thm/8d1952ba2b3c6dcd76236f090ab8642c` to `http://marketing.nahamstore.thm/8d1952ba2b3c6dcd76236f090ab8642a`
     this gives me an error `/?error=Campaign+Not+Found`, so i tried to add simple payload for XSS vuln `/?error=<script>alert(1);</script>`, and it gives me the alert popup, so I confirm it have XSS vuln
    - I tried also the `/search?q=<script>alert('1');</script>`, found searchbox in the homepage, it doesn't work because in the script input found in the html element filter it, I use `dalfox url <target_domain>` to confirm for possible payload.
    - I found also the `/product?id=` can use the payload, so I tried SQL Injection, `?id=0 UNION SELECT 1,2,3,4,5-- -` found it have 5 columns required and 2 tables (as per task 10)


### Task 3: Open Redirect

- Upon navigating to each of the endpoint I found `redirect_url=/basket`, so it's from basket I click add address, I confirm it's redirecting when I changed the endpoint to e.g. `https://google.com` upon adding test address and information.

### Task 4: CSRF

- I found the `csrf_protect` input variable hidden that is in base64, and I found it in `/account/settings/email` when i want to change email but the password change is not protected.
- the value in the `csrf_protect` is base64, so I use cyberchef to get the string value, until I get some useful information of user_id and timestamp

### Task 5: IDOR

I use Burp here to analyze the request or directly edit form in the html
- so in `/basket`, i can change the `address_id input` and send it so it shows different user and address in my basket, then I can successfully add it to order without checking if its under my user id.

### Task 6: SSRF

### Task 7: LFI

### Task 10: SQL Injection

- using `sqlmap` to do sql injection to find the flag 1

