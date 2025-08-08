# recon-kapsaml-rehber


# Bug Hunting Recon Methodology - Kapsamlı Rehber

## 📋 İçindekiler
1. [Recon Nedir ve Neden Önemli?](#recon-nedir)
2. [Recon Aşamaları](#recon-aşamaları)
3. [Pasif Recon Teknikleri](#pasif-recon)
4. [Aktif Recon Teknikleri](#aktif-recon)
5. [Araçlar ve Teknolojiler](#araçlar)
6. [Pratik Trickler ve İpuçları](#trickler)
7. [Otomasyon ve Scriptler](#otomasyon)
8. [Gerçek Dünya Senaryoları](#senaryolar)

---

## 🎯 Recon Nedir ve Neden Önemli? {#recon-nedir}

**Reconnaissance (Keşif)**, hedef hakkında bilgi toplama sürecidir. Bug hunting'de başarının %80'i iyi bir recon'a dayanır.

### Recon'un Amacı:
- **Attack Surface'i genişletmek** (saldırı yüzeyini büyütmek)
- **Gizli endpoint'ler** bulmak
- **Subdomain'ler** keşfetmek
- **Teknoloji stack'ini** anlamak
- **Potansiyel entry point'leri** belirlemek

---

## 🔄 Recon Aşamaları {#recon-aşamaları}

### 1. Planlama ve Hedef Belirleme
```
Target: example.com
Scope: *.example.com (wildcard subdomain dahil)
Out of Scope: third-party domains
Time Frame: 3-5 gün
```

### 2. Bilgi Toplama Hiyerarşisi
```
Level 1: Domain/Subdomain Discovery
Level 2: Port/Service Enumeration
Level 3: Web Technology Identification
Level 4: Content Discovery
Level 5: Parameter & Endpoint Discovery
```

---

## 🕵️ Pasif Recon Teknikleri {#pasif-recon}

### A) Domain/Subdomain Discovery

#### 1. Certificate Transparency Logs
```bash
# crt.sh sorgusu
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u

# Certspotter
curl -s "https://api.certspotter.com/v1/issuances?domain=target.com&include_subdomains=true&expand=dns_names" | jq -r '.[].dns_names[]' | sort -u
```

#### 2. DNS Enumeration
```bash
# Subfinder kullanımı
subfinder -d target.com -all -recursive

# Amass kullanımı
amass enum -d target.com -config amass_config.ini

# Manual DNS queries
dig target.com ANY
dig target.com TXT
dig target.com MX
```

#### 3. Search Engine Dorking
```
Google Dorks:
site:target.com -www
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com intitle:"index of"
inurl:target.com -site:target.com

Bing/Yahoo:
domain:target.com
ip:192.168.1.1

Shodan:
hostname:target.com
ssl.cert.subject.CN:target.com
```

#### 4. Archive & Historical Data
```bash
# Wayback Machine
curl -s "http://web.archive.org/cdx/search/cdx?url=*.target.com/*&output=text&fl=original&collapse=urlkey" | sort -u

# GitHub/GitLab Search
# Manuel olarak: site:github.com "target.com"
# GitDorker tool ile otomatik
```

### B) OSINT Kaynaklarından Bilgi Toplama

#### 1. Whois Information
```bash
whois target.com
whois -h whois.radb.net target.com
```

#### 2. Social Media & Public Records
```
LinkedIn: çalışanlar, teknolojiler
Twitter: duyurular, yeni özellikler  
Job Postings: kullanılan teknolojiler
Company Blog: sistem mimarisi
```

---

## 🎯 Aktif Recon Teknikleri {#aktif-recon}

### A) Port & Service Scanning

#### 1. Nmap Scans
```bash
# Hızlı port tarama
nmap -T4 -p- --min-rate=1000 target.com

# Servis detection
nmap -sC -sV -p 22,80,443,8080 target.com

# UDP scan (önemli servisler için)
nmap -sU -p 53,161,500 target.com

# Stealth scan
nmap -sS -f -T2 target.com
```

#### 2. Masscan ile Hızlı Tarama
```bash
masscan -p1-65535 target.com --rate=1000 -e tun0 --router-ip 10.0.0.1
```

### B) Web Application Discovery

#### 1. Directory & File Brute Force
```bash
# Dirsearch
python3 dirsearch.py -u https://target.com -e php,asp,aspx,jsp,html,js -t 50

# Gobuster
gobuster dir -u https://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,js,txt,xml

# Custom wordlist ile
gobuster dir -u https://target.com -w custom_endpoints.txt -s 200,204,301,302,307,403,500
```

#### 2. Parameter Discovery
```bash
# Arjun
python3 arjun.py -u https://target.com/page

# ParamSpider
python3 paramspider.py --domain target.com

# Manual testing
# GET parameters: ?id=1&user=admin&debug=true
# POST parameters: Burp Suite ile brute force
```

#### 3. JavaScript Analysis
```bash
# LinkFinder - JS dosyalarından endpoint çıkarma
python linkfinder.py -i https://target.com/app.js -o cli

# JSParser
python3 jsparser.py -u https://target.com

# Manual JS Review
# Developer tools > Sources > Search
```

### C) Advanced Discovery Techniques

#### 1. Virtual Host Discovery
```bash
# Gobuster vhost
gobuster vhost -u target.com -w subdomains.txt

# Manual host header manipulation
curl -H "Host: admin.target.com" https://target.com
```

#### 2. API Discovery
```bash
# Common API paths
/api/v1/
/api/v2/
/rest/
/graphql
/swagger.json
/openapi.json

# API fuzzing
ffuf -w api_wordlist.txt -u https://target.com/FUZZ -mc 200,201,400,401,403,500
```

---

## 🛠️ Araçlar ve Teknolojiler {#araçlar}

### Essential Tools

#### 1. Subdomain Discovery
```bash
# Pasif tools
Subfinder, Amass, Assetfinder, Findomain

# Aktif tools  
Gobuster, Massdns, Shuffledns

# Combined approach
cat passive_subs.txt active_subs.txt | sort -u | httpx -mc 200,201,301,302,403,500
```

#### 2. HTTP Analysis
```bash
# HTTPx - HTTP probe
cat subdomains.txt | httpx -title -tech-detect -status-code

# Nuclei - Vulnerability scanner
nuclei -l urls.txt -t nuclei-templates/

# FFUF - Fast fuzzer
ffuf -w wordlist.txt -u https://target.com/FUZZ -mc 200,204,301,302,307,403,500
```

#### 3. Information Gathering
```bash
# Whatweb - Technology identification
whatweb https://target.com

# Wappalyzer CLI
wappalyzer https://target.com

# Retire.js - JavaScript library vulnerabilities
retire --js --outputformat json https://target.com
```

---

## 💡 Pratik Trickler ve İpuçları {#trickler}

### A) Advanced Subdomain Discovery

#### 1. Permutation Attacks
```bash
# Altdns - subdomain permutation
python altdns.py -i subdomains.txt -o permuted_subdomains.txt -w words.txt
cat permuted_subdomains.txt | massdns -r resolvers.txt -t A -o S -w results.txt
```

#### 2. DNS Brute Force with Smart Wordlists
```bash
# Birleştirulmiş wordlist oluşturma
cat /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt > combined_wordlist.txt
cat company_specific_keywords.txt >> combined_wordlist.txt

# Massdns ile brute force
massdns -r resolvers.txt -t A combined_wordlist.txt -o S | cut -d' ' -f1 | sed 's/\.$//' | sort -u
```

#### 3. Certificate Transparency Advanced Queries
```bash
# Multiple CT logs query
for log in $(cat ct_logs.txt); do
    curl -s "$log/ct/v1/get-entries?start=0&end=1000" | jq -r '.entries[].leaf_input' | base64 -d | grep -o 'target\.com' | head -20
done
```

### B) Content Discovery Tricks

#### 1. Extension-based Fuzzing
```bash
# Belirli extension'lar için özel wordlist
echo -e ".bak\n.old\n.tmp\n.backup\n.orig\n.save" > extensions.txt

# Her endpoint için extension dene
while read url; do
    while read ext; do
        curl -s -o /dev/null -w "%{http_code}" "$url$ext" | grep -E "200|301|302"
    done < extensions.txt
done < endpoints.txt
```

#### 2. HTTP Method Fuzzing
```bash
# Farklı HTTP method'ları test et
for method in GET POST PUT DELETE PATCH OPTIONS HEAD; do
    curl -X $method -s -o /dev/null -w "$method: %{http_code}\n" https://target.com/api/users
done
```

#### 3. Parameter Pollution Testing
```bash
# Parameter pollution
curl "https://target.com/search?q=test&q=admin"
curl "https://target.com/api?user=normal&user=admin"

# Array parameter testing  
curl "https://target.com/api?users[]=1&users[]=2&users[admin]=true"
```

### C) JavaScript Analysis Advanced

#### 1. Source Map Discovery
```bash
# .map dosyası arama
curl -s https://target.com/app.js | grep -o 'sourceMappingURL=.*\.map'

# Map file analysis
curl -s https://target.com/app.js.map | jq '.sources[]' | grep -v node_modules
```

#### 2. Webpack Bundle Analysis
```bash
# Webpack chunk'ları bulma
curl -s https://target.com | grep -oP 'chunk\.\w+\.js' | sort -u

# Her chunk'ı analiz et
for chunk in $(cat chunks.txt); do
    echo "=== Analyzing $chunk ==="
    curl -s "https://target.com/$chunk" | js-beautify | grep -E "(api|endpoint|url|path)"
done
```

---

## 🤖 Otomasyon ve Scriptler {#otomasyon}

### A) Recon Pipeline Script

```bash
#!/bin/bash
# recon_pipeline.sh

TARGET=$1
OUTPUT_DIR="recon_$TARGET"

echo "[+] Starting recon for $TARGET"
mkdir -p $OUTPUT_DIR

# 1. Subdomain discovery
echo "[+] Subdomain discovery..."
subfinder -d $TARGET -all -o $OUTPUT_DIR/subdomains_passive.txt
amass enum -d $TARGET -o $OUTPUT_DIR/subdomains_amass.txt
cat $OUTPUT_DIR/subdomains_*.txt | sort -u > $OUTPUT_DIR/subdomains_all.txt

# 2. HTTP probing
echo "[+] HTTP probing..."
cat $OUTPUT_DIR/subdomains_all.txt | httpx -mc 200,201,301,302,403,500 -o $OUTPUT_DIR/live_hosts.txt

# 3. Technology detection
echo "[+] Technology detection..."
cat $OUTPUT_DIR/live_hosts.txt | httpx -tech-detect -o $OUTPUT_DIR/tech_stack.txt

# 4. Directory brute force
echo "[+] Directory discovery..."
while read host; do
    gobuster dir -u $host -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o $OUTPUT_DIR/dirs_$(echo $host | cut -d'/' -f3).txt -q
done < $OUTPUT_DIR/live_hosts.txt

# 5. Vulnerability scanning
echo "[+] Vulnerability scanning..."
nuclei -l $OUTPUT_DIR/live_hosts.txt -t nuclei-templates/ -o $OUTPUT_DIR/vulns.txt

echo "[+] Recon completed! Check $OUTPUT_DIR directory"
```

### B) Continuous Monitoring Script

```bash
#!/bin/bash
# monitor_target.sh

TARGET=$1
PREVIOUS_SUBS="previous_subdomains.txt"
CURRENT_SUBS="current_subdomains.txt"

# Mevcut subdomain'leri bul
subfinder -d $TARGET -all | sort -u > $CURRENT_SUBS

# Yeni subdomain kontrol et
if [ -f $PREVIOUS_SUBS ]; then
    NEW_SUBS=$(comm -13 $PREVIOUS_SUBS $CURRENT_SUBS)
    if [ ! -z "$NEW_SUBS" ]; then
        echo "[!] New subdomains found:"
        echo "$NEW_SUBS"
        
        # Yeni subdomain'leri otomatik test et
        echo "$NEW_SUBS" | httpx -mc 200,201,301,302,403,500 | nuclei -t nuclei-templates/
    fi
fi

cp $CURRENT_SUBS $PREVIOUS_SUBS
```

---

## 🌍 Gerçek Dünya Senaryoları {#senaryolar}

### Senaryo 1: E-commerce Sitesi Recon

```
Target: shop.example.com
Objective: Find admin panels, API endpoints, user data exposure

Step 1: Subdomain discovery
- api.shop.example.com
- admin.shop.example.com  
- dev.shop.example.com
- staging.shop.example.com

Step 2: Content discovery
- /admin/login.php (403 Forbidden)
- /api/v1/users (200 OK - Sensitive!)
- /backup/database.sql (403 Forbidden)

Step 3: Parameter discovery  
- /api/v1/users?user_id=1&admin=true
- /search?q=test&debug=1&internal=true

Critical Findings:
- API endpoint without authentication
- Debug parameters in production
- Admin panel accessible via specific User-Agent
```

### Senaryo 2: SaaS Platform Recon

```
Target: app.saas-company.com
Challenge: Multi-tenant architecture

Subdomain pattern discovered:
- customer1.app.saas-company.com
- customer2.app.saas-company.com  
- admin.app.saas-company.com

Technique: Subdomain permutation
- Generated 50,000+ possible tenant names
- Found 500+ active tenant subdomains
- Discovered leaked customer data via misconfigured S3 buckets

Key insight: Company naming patterns
- Fortune 500 company names as subdomains
- Some tenants had default credentials
```

### Senaryo 3: API-First Company

```
Target: api.techcompany.com
Focus: GraphQL ve REST API discovery

Discovery process:
1. Found /graphql endpoint via robots.txt
2. Introspection query revealed internal schemas
3. Found deprecated v1 API still accessible
4. Mobile app traffic analysis revealed hidden endpoints

GraphQL introspection:
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    types { ...FullType }
  }
}

Result: 
- Found admin mutations in production GraphQL
- v1 API had SQL injection vulnerability
- Rate limiting bypass via different API versions
```

---

## 🎓 Pro Tips ve Best Practices

### 1. Recon Methodology Principles

**Always Follow the Data Flow:**
```
User Input → Frontend → API → Database → External Services
```

**Think Like a Developer:**
- Staging/dev environments
- Backup files with timestamps
- Debug endpoints left in production
- Default configurations

**Automation + Manual = Success:**
- Otomatik araçlar bulk data için
- Manual testing unique patterns için
- Always verify automated findings

### 2. Common Mistakes to Avoid

❌ **Sadece otomatik araçlara güvenmek**
✅ Manual verification ve custom wordlists

❌ **Scope'u daralt tutmak**  
✅ Wildcard subdomains ve related domains

❌ **Eski data'yı silmek**
✅ Historical comparison için saklamak

❌ **Single source ile yetinmek**
✅ Multiple sources ve cross-validation

### 3. Time Management

```
Day 1-2: Passive recon (80% effort)
Day 3: Active scanning (15% effort)  
Day 4-5: Manual verification ve deep dive (5% effort)

Ratio: 80% discovery, 20% exploitation
```

### 4. Documentation

```
## Recon Results - Target: example.com
### Date: 2024-XX-XX

**Subdomains Found:** 450
**Live Hosts:** 320  
**Technologies Identified:** 
- WordPress 5.8.1
- Apache 2.4.41
- MySQL 8.0

**Critical Findings:**
1. Admin panel at admin.example.com (default creds)
2. API endpoint leaking user data
3. Backup files in /old/ directory

**Next Steps:**
- Test admin panel authentication
- Analyze API endpoints for IDOR
- Check backup files for sensitive data
```

---

## 🎯 Final Checklist

### Before Starting Recon:
- [ ] Scope belirlenmiş mi?
- [ ] Legal authorization var mı?
- [ ] Tools hazır mı?
- [ ] Output directory oluşturuldu mu?

### During Recon:
- [ ] Multiple sources kullanılıyor mu?
- [ ] Results dokümante ediliyor mu?
- [ ] False positives filtreleniyor mu?
- [ ] Rate limiting respect ediliyor mu?

### After Recon:
- [ ] Results organize edildi mi?
- [ ] Critical findings prioritize edildi mi?  
- [ ] Next steps planlandı mı?
- [ ] Backup alındı mı?

---

Bu rehber, bug hunting recon methodology'sinin tüm aspektlerini kapsar. Her bölümü uygulamalı olarak çalış ve kendi experience'ına göre customize et. Unutma ki iyi bir recon, successful bug hunting'in temelidir!

**"The best hackers are not necessarily the best programmers, but they are the best at reconnaissance."** 

Practice makes perfect - bu metodoloji ile daily practice yap ve kendi unique approach'ını geliştir!
