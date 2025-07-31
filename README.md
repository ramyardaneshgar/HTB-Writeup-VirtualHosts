# VirtualHosts
Virtual Host Enumeration using Gobuster to identify hidden subdomains and configurations.

---

#### **Background**
The goal of this lab was to identify hidden subdomains hosted on `inlanefreight.htb` using virtual host (VHost) enumeration. Virtual hosting enables web servers to host multiple domains or subdomains on the same IP address by leveraging the HTTP `Host` header. By brute-forcing VHosts, I aimed to uncover subdomains prefixed with "web", "vm", "br", "a", and "su".

---

#### **Steps I Took**

1. **Environment Setup**  
   - I started by ensuring that the domain `inlanefreight.htb` resolved locally. This involved mapping the target IP to the domain in my `/etc/hosts` file:
     ```bash
     sudo sh -c "echo '94.237.50.3 inlanefreight.htb' >> /etc/hosts"
     ```

2. **Running Gobuster for VHost Enumeration**  
   - I used **Gobuster** in VHOST enumeration mode with a commonly used wordlist from the SecLists repository:
     ```bash
     gobuster vhost -u http://inlanefreight.htb:56255 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain -t 10
     ```
   - This setup allowed Gobuster to append the base domain (`inlanefreight.htb`) to each word in the wordlist and test it against the target server.

3. **Analyzing Gobuster Output**  
   - Gobuster identified several valid subdomains:
     ```
     Found: blog.inlanefreight.htb:56255 Status: 200 [Size: 98]
     Found: forum.inlanefreight.htb:56255 Status: 200 [Size: 100]
     Found: admin.inlanefreight.htb:56255 Status: 200 [Size: 100]
     Found: support.inlanefreight.htb:56255 Status: 200 [Size: 104]
     Found: vm5.inlanefreight.htb:56255 Status: 200 [Size: 96]
     Found: browse.inlanefreight.htb:56255 Status: 200 [Size: 102]
     Found: web17611.inlanefreight.htb:56255 Status: 200 [Size: 106]
     ```

4. **Mapping Subdomains to Required Prefixes**  
   - Based on the Gobuster output, I mapped the subdomains to the given prefixes:
     - **"web"** → `web17611.inlanefreight.htb`
     - **"vm"** → `vm5.inlanefreight.htb`
     - **"br"** → `browse.inlanefreight.htb`
     - **"a"** → `admin.inlanefreight.htb`
     - **"su"** → **Not Found**

5. **Targeting the Missing Prefix "su"**  
   - Since "su" was not found in the initial scan, I created a custom wordlist with "su" as the prefix:
     ```bash
     echo "su" > su_prefix_wordlist.txt
     gobuster vhost -u http://inlanefreight.htb:56255 -w su_prefix_wordlist.txt --append-domain -t 10
     ```
   - Unfortunately, this scan also did not yield results for the "su" prefix.

6. **Verifying Results**  
   - To confirm the identified subdomains, I added them to `/etc/hosts` and tested their responses:
     ```bash
     sudo sh -c "echo '94.237.50.3 web17611.inlanefreight.htb' >> /etc/hosts"
     curl -I http://web17611.inlanefreight.htb:56255
     ```

---

#### **Lessons Learned**

1. **Effective Use of Wordlists**  
   The choice of wordlist significantly impacts the success of VHost enumeration. SecLists provided a robust foundation for discovery, but targeted custom wordlists can fill gaps.

2. **Iterative Testing**  
   Combining broader scans with focused, custom scans (e.g., for "su") is an effective strategy when the initial output is incomplete.

3. **Understanding Virtual Hosting**  
   I deepened my understanding of how web servers distinguish between domains using HTTP `Host` headers and VHost configurations.

4. **Tool Proficiency**  
   Gobuster proved to be an efficient tool for VHost enumeration, especially with its flexibility to handle large wordlists and custom domains.

---
