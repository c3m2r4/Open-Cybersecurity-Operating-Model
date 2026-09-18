Absolutely. What you discovered is that the HBO page is **not using the old `/game-of-thrones/cast-and-crew/<slug>` structure anymore**. The old URL redirects to HBO Max, but the returned HTML still contains the character data in the page's serialized Next.js/application data.

Your successful extraction:

```text
97 users
alliser-thorne
archmaester-ebrose
arya-stark
...
```

is therefore the right foundation.

However, because the next step appears to be using these names against your **GOAD/Exchange lab**, I’ll frame the runbook around **authorized lab username discovery and validation**, not password spraying against HBO or other real-world services.

# Game of Thrones → GOAD Username Discovery Runbook

## 1. Objective

Build a repeatable workflow that:

1. Retrieves the Game of Thrones cast/character page.
    
2. Follows the HBO → HBO Max redirect.
    
3. Identifies the application's embedded character data.
    
4. Extracts the `urlSlug` values.
    
5. Normalizes the names.
    
6. Removes duplicates.
    
7. Produces a clean `users.txt`.
    
8. Compares those candidate usernames against your **GOAD Active Directory**.
    
9. Uses only confirmed lab accounts for subsequent authentication testing.
    

### Lab architecture

```text
                    Internet
                       │
                       ▼
              HBO/HBO Max webpage
                       │
                       │ HTTPS
                       ▼
                 /tmp/got.html
                       │
                       ▼
              Extract urlSlug
                       │
                       ▼
                  users.txt
                       │
                       ▼
             Candidate usernames
                       │
                       ▼
             ┌─────────────────┐
             │   GOAD AD Lab   │
             │ sevenkingdoms   │
             └─────────────────┘
                       │
                       ▼
             Get-ADUser validation
                       │
                       ▼
             Confirmed lab users
```

---

# Phase 1 — Prepare Kali

Create a dedicated working directory instead of keeping everything in `~`.

```bash
mkdir -p ~/labs/goad/got-user-enum
cd ~/labs/goad/got-user-enum
```

Verify the tools:

```bash
which curl
which grep
which sed
which sort
which uniq
which awk
```

Optional:

```bash
curl --version
```

---

# Phase 2 — Verify the original HBO URL

Run:

```bash
curl -I https://www.hbo.com/game-of-thrones/cast-and-crew
```

You should see something similar to:

```text
HTTP/2 301
location: https://www.hbomax.com/show/.../cast-and-crew
```

### Interpretation

The important part is:

```text
HTTP/2 301
```

and:

```text
location: https://www.hbomax.com/...
```

That means HBO is telling the client:

> "This resource has moved."

Do **not** assume the old HBO URL is still the page you're parsing.

---

# Phase 3 — Download the redirected page

Use:

```bash
curl -L -A 'Mozilla/5.0' -s \
  https://www.hbo.com/game-of-thrones/cast-and-crew \
  -o got.html
```

Check the file:

```bash
ls -lh got.html
```

Then:

```bash
wc -c got.html
```

You previously received approximately:

```text
478147
```

which confirms that you're getting substantial HTML rather than an empty redirect response.

---

# Phase 4 — Establish what the page contains

Before writing an extraction command, inspect it.

### Search for the page title

```bash
grep -oi 'Game of Thrones Cast[^<]*' got.html | head
```

Or:

```bash
grep -oiE '.{0,100}Game of Thrones.{0,150}' got.html | head -20
```

You should find:

```text
Game of Thrones Cast & Characters | HBO Max
```

---

# Phase 5 — Identify the character structure

Search for known characters:

```bash
grep -oiE '.{0,100}(Jon Snow|Tyrion|Daenerys|Arya Stark|Cersei).{0,150}' \
  got.html | head -30
```

You discovered that the HTML contains structures such as:

```text
primaryText":"Daenerys Targaryen"
urlSlug":"daenerys-targaryen"
```

and:

```text
primaryText":"Jon Snow"
urlSlug":"jon-snow"
```

This is the important discovery.

The application isn't necessarily exposing the information through the visible HTML attributes you initially expected.

---

# Phase 6 — Understand why your first extraction returned 0

Your first command looked for:

```bash
href="/game-of-thrones/cast-and-crew/
```

But the current HTML actually contains:

```text
href="/show/4f6b4985-2dc9-4ab6-ac79-d60f0860b0ac/cast-and-crew/jon-snow"
```

Therefore:

```bash
grep 'href="/game-of-thrones/cast-and-crew/'
```

returns nothing.

Consequently:

```text
0 users
```

was expected.

### Lesson

When scraping a changing website, don't immediately assume the URL structure from an old version of the site.

First establish:

```text
HTTP response
       ↓
redirect
       ↓
actual page
       ↓
HTML structure
       ↓
embedded data
       ↓
extraction method
```

---

# Phase 7 — Extract the `urlSlug` values

Your successful extraction was:

```bash
grep -oE 'urlSlug\\?":\\?"[^"]+' got.html |
sed -E 's/.*urlSlug\\?":\\?"//' |
sort -u > users.txt
```

This extracts strings such as:

```text
daenerys-targaryen
tyrion-lannister
jon-snow
jaime-lannister
cersei-lannister
```

Check the result:

```bash
wc -l users.txt
```

You got:

```text
97 users.txt
```

Then:

```bash
head -30 users.txt
```

---

# Phase 8 — Validate the extracted data

Don't immediately use the file.

First inspect it.

### Count lines

```bash
wc -l users.txt
```

### Check for duplicates

```bash
sort users.txt | uniq -d
```

No output means there are no duplicate lines.

### Check for blank lines

```bash
grep -n '^$' users.txt
```

### Check for whitespace

```bash
grep -nE '[[:space:]]' users.txt
```

### Check for unexpected characters

```bash
grep -nEv '^[a-z0-9-]+$' users.txt
```

The last command should produce nothing if every entry is a clean lowercase slug.

---

# Phase 9 — Normalize the usernames

If your intended lab naming convention converts hyphens to periods, create a **separate candidate file** rather than destroying the original.

Keep:

```text
users.txt
```

as the source file.

Create:

```bash
sed 's/-/./g' users.txt | sort -u > users-dot.txt
```

For example:

```text
arya-stark
```

becomes:

```text
arya.stark
```

And:

```text
jon-snow
```

becomes:

```text
jon.snow
```

Check:

```bash
head -30 users-dot.txt
```

---

# Phase 10 — Keep multiple representations

For a clean lab workflow, maintain:

```text
got.html
users.txt
users-dot.txt
```

I recommend:

```text
got.html
    │
    └── source webpage

users.txt
    │
    └── original HBO slugs

users-dot.txt
    │
    └── normalized candidate usernames
```

This makes troubleshooting much easier.

---

# Phase 11 — Compare candidates against GOAD AD

This is where the workflow becomes useful for your GOAD environment.

On the Windows domain controller, first retrieve your actual AD users.

For example:

```powershell
Get-ADUser -Filter * |
    Select-Object Name,SamAccountName,UserPrincipalName,Enabled |
    Sort-Object SamAccountName
```

Export the usernames:

```powershell
Get-ADUser -Filter * |
    Select-Object -ExpandProperty SamAccountName |
    Set-Content C:\Temp\ad-users.txt
```

You now have:

```text
C:\Temp\ad-users.txt
```

---

# Phase 12 — Compare your candidate list with AD

Transfer the authorized AD username list to Kali.

For example:

```text
~/labs/goad/got-user-enum/ad-users.txt
```

Normalize it:

```bash
tr '[:upper:]' '[:lower:]' < ad-users.txt |
sort -u > ad-users-clean.txt
```

Then compare:

```bash
comm -12 \
  <(sort users-dot.txt) \
  <(sort ad-users-clean.txt)
```

This gives you the intersection:

```text
candidate username
        ∩
actual GOAD account
```

That is much more valuable than assuming that every Game of Thrones character corresponds to an AD account.

---

# Phase 13 — Produce the confirmed lab-user list

Save the intersection:

```bash
comm -12 \
  <(sort users-dot.txt) \
  <(sort ad-users-clean.txt) \
  > confirmed-users.txt
```

Check:

```bash
wc -l confirmed-users.txt
```

Then:

```bash
cat confirmed-users.txt
```

Your workflow now becomes:

```text
HBO character data
        ↓
97 candidate slugs
        ↓
normalize
        ↓
candidate usernames
        ↓
compare with AD
        ↓
confirmed GOAD accounts
```

---

# Phase 14 — Validate accounts from Windows

On your domain controller:

```powershell
Get-ADUser -Filter * |
    Where-Object {
        $_.SamAccountName -in (Get-Content C:\Temp\confirmed-users.txt)
    } |
    Select-Object Name,
                  SamAccountName,
                  UserPrincipalName,
                  Enabled |
    Format-Table -AutoSize
```

This is the key validation step.

You want to distinguish:

```text
Candidate
```

from:

```text
Confirmed AD account
```

---

# Phase 15 — Create a reusable extraction script

Instead of manually repeating the commands, create:

```bash
nano extract-got-users.sh
```

Use:

```bash
#!/usr/bin/env bash

set -euo pipefail

URL="https://www.hbo.com/game-of-thrones/cast-and-crew"
HTML="got.html"
USERS="users.txt"
NORMALIZED="users-dot.txt"

echo "[+] Downloading page..."

curl -L \
    -A 'Mozilla/5.0' \
    -sS \
    "$URL" \
    -o "$HTML"

echo "[+] Extracting urlSlug values..."

grep -oE 'urlSlug\\?":\\?"[^"]+' "$HTML" |
    sed -E 's/.*urlSlug\\?":\\?"//' |
    sort -u > "$USERS"

echo "[+] Creating normalized candidate usernames..."

sed 's/-/./g' "$USERS" |
    sort -u > "$NORMALIZED"

echo
echo "[+] Results"
echo "    Raw candidates : $(wc -l < "$USERS")"
echo "    Normalized     : $(wc -l < "$NORMALIZED")"
echo
echo "[+] First 20:"
head -20 "$NORMALIZED"
```

Make it executable:

```bash
chmod +x extract-got-users.sh
```

Run:

```bash
./extract-got-users.sh
```

---

# Phase 16 — Add sanity checks

A good runbook should fail safely if the webpage changes.

Add:

```bash
if [ ! -s "$USERS" ]; then
    echo "[!] Extraction returned zero users."
    echo "[!] The webpage structure may have changed."
    exit 1
fi
```

You can also require a reasonable minimum:

```bash
COUNT=$(wc -l < "$USERS")

if [ "$COUNT" -lt 10 ]; then
    echo "[!] Suspiciously small result: $COUNT"
    exit 1
fi
```

This prevents accidentally treating an error page as a valid username list.

---

# Troubleshooting Runbook

## Problem 1 — `users.txt` contains 0 lines

Run:

```bash
wc -c got.html
```

Then:

```bash
grep -oE 'urlSlug[^,}]*' got.html | head
```

If you get results, your extraction regex needs adjustment.

---

## Problem 2 — HBO returns 301

Expected.

Use:

```bash
curl -L
```

The `-L` tells curl to follow redirects.

---

## Problem 3 — The page is empty

Check:

```bash
curl -IL https://www.hbo.com/game-of-thrones/cast-and-crew
```

Then:

```bash
curl -L -A 'Mozilla/5.0' \
  https://www.hbo.com/game-of-thrones/cast-and-crew |
  wc -c
```

---

## Problem 4 — `urlSlug` disappears

Search for the known character:

```bash
grep -oi 'Jon Snow' got.html | head
```

Then:

```bash
grep -oE '.{0,100}Jon Snow.{0,200}' got.html | head
```

The application may have changed its serialization format.

---

## Problem 5 — Candidate doesn't exist in AD

That's normal.

A character slug such as:

```text
jon-snow
```

doesn't prove that:

```text
jon.snow
```

exists in your AD.

Always validate with:

```powershell
Get-ADUser
```

---

# Recommended Directory Layout

For your GOAD work, I'd structure this as:

```text
~/labs/goad/
└── got-user-enum/
    ├── got.html
    ├── users.txt
    ├── users-dot.txt
    ├── ad-users.txt
    ├── ad-users-clean.txt
    ├── confirmed-users.txt
    └── extract-got-users.sh
```

And keep your evidence separated:

```text
raw/
    got.html

candidates/
    users.txt
    users-dot.txt

validated/
    ad-users-clean.txt
    confirmed-users.txt

scripts/
    extract-got-users.sh
```

That makes the exercise look much more like a professional penetration-testing workflow.

---

# Final Runbook

|Phase|Action|Output|
|---|---|---|
|1|Request original HBO page|301|
|2|Follow redirect|HBO Max page|
|3|Save HTML|`got.html`|
|4|Identify embedded data|`urlSlug`|
|5|Extract slugs|`users.txt`|
|6|Deduplicate|97 candidates|
|7|Normalize|`users-dot.txt`|
|8|Obtain authorized AD usernames|`ad-users.txt`|
|9|Normalize AD data|`ad-users-clean.txt`|
|10|Compare lists|confirmed accounts|
|11|Validate with `Get-ADUser`|verified GOAD identities|
|12|Use only verified lab identities for further testing|controlled lab exercise|

### The important lesson from your troubleshooting

Your initial approach was based on the **old URL/HTML structure**:

```text
/game-of-thrones/cast-and-crew/<name>
```

The current page instead contains:

```text
/show/<show-id>/cast-and-crew/<slug>
```

and, more importantly, serialized application data containing:

```text
primaryText
secondaryText
urlSlug
```

So the robust approach is:

```text
Don't scrape what you expect.
        ↓
Download what the server actually returns.
        ↓
Inspect the structure.
        ↓
Identify stable data fields.
        ↓
Extract those fields.
        ↓
Validate against the authorized target.
```

That methodology will transfer nicely to your **GOAD + Exchange lab** work.