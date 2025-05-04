Thought for a couple of seconds


# Localization Refresher Notes

## 1. Purpose of Localization

* **Goal:** Let users and administrators work in their native language and regional conventions (e.g. Canada/English vs Canada/French) without manual conversions.
* **Scope:** Affects command-line programs, GUIs, date/time formats, number and currency formatting, collation, messages, etc.

---

## 2. Locale Environment Variables

Linux programs read locale settings from environment variables. They are, in order of precedence:

| Variable      | Purpose                                                                          | Example                                        |
| ------------- | -------------------------------------------------------------------------------- | ---------------------------------------------- |
| `LC_ALL`      | **Override all** other `LC_*` and `LANG`; use this single locale for everything. | `export LC_ALL="fr_CA.UTF-8"`                  |
| `LC_CTYPE`    | Character classification and encoding.                                           | `export LC_CTYPE="en_US.UTF-8"`                |
| `LC_COLLATE`  | Collation order for string compare/sort.                                         | `export LC_COLLATE="en_US.UTF-8"`              |
| `LC_TIME`     | Date and time formats.                                                           | `export LC_TIME="en_US.UTF-8"`                 |
| `LC_NUMERIC`  | Decimal point and thousands separator.                                           | `export LC_NUMERIC="en_US.UTF-8"`              |
| `LC_MONETARY` | Currency formats (symbol, placement).                                            | `export LC_MONETARY="en_US.UTF-8"`             |
| `LC_MESSAGES` | Language for printed messages and parsing program responses.                     | `export LC_MESSAGES="en_US.UTF-8"`             |
| `LANG`        | Fallback locale when no `LC_*` is set.                                           | `export LANG="en_US.UTF-8"`                    |
| `NLSPATH`     | Directories for message catalogs used by `catopen()`.                            | `export NLSPATH="/usr/share/locale/%L/%N.cat"` |

**Notes:**

* Unset an `LC_*` to let `LANG` take effect.
* Setting `LC_ALL` is a quick way to override everything, but not recommended for persistent use.

---

## 3. Character Encodings

### 3.1 ASCII

* **7-bit**, 128 characters:

  * 95 **printable** (A–Z, a–z, 0–9, punctuation, space)
  * 33 **control** codes (e.g. newline, tab).
* **Command:**

  ```bash
  ascii              # interactive table of ASCII codes
  ```

### 3.2 Unicode & UTF-8

* **Unicode:** Assigns a unique code point to every character in every language.
* **UTF-8:** Variable-width encoding (1–4 bytes) for Unicode, **backward-compatible with ASCII**.
* **Activate UTF-8 locale:**

  ```bash
  export LANG="en_US.UTF-8"
  locale              # verify all LC_* show .UTF-8
  ```

### 3.3 ISO/IEC 8859

* **8-bit extensions** to ASCII, multiple parts (e.g. ISO-8859-1 for Western European languages).
* Uses high bit for extra 96 printable characters (accents, symbols).

---

## 4. Converting Between Encodings

* **`iconv`** – convert text files from one encoding to another.

  * **List supported encodings:**

    ```bash
    iconv -l
    ```
  * **Convert file:**

    ```bash
    iconv -f ISO-8859-1 -t CP437 old.txt > new.txt
    ```

---

## 5. Time Zone, Date & Time

### 5.1 System vs User Time Zones

* **System clock:** Always in **UTC** internally.
* **Local presentation:** Determined by TZ environment or system config.

### 5.2 User Time Zone (`TZ` variable)

* **Temporarily set for a session:**

  ```bash
  export TZ="America/Los_Angeles"
  date       # now shows PST/PDT
  ```
* **Persist for one user:**

  ```bash
  # in ~/.profile or ~/.bash_profile
  TZ="America/Los_Angeles"; export TZ
  ```

### 5.3 System Time Zone Files

* **Binary config:** `/etc/localtime` (do **not** edit directly; back it up before changes).
* **Distribution-specific pointers:**

  * **Debian/Ubuntu:** `/etc/timezone` (text, e.g. `Etc/UTC`)
  * **RHEL/CentOS:** `/etc/sysconfig/clock` (e.g. `ZONE="America/Los_Angeles"`)

### 5.4 Time Zone Data Directory

* Contains zone files under `/usr/share/zoneinfo`.

  ```bash
  ls /usr/share/zoneinfo           # lists continents and regions
  ls /usr/share/zoneinfo/Europe    # e.g. London, Berlin, etc.
  ```

### 5.5 Interactive Selection with `tzselect`

1. Run `tzselect`.
2. Choose **continent/ocean** (e.g. “2) Americas”).
3. Choose **country** (e.g. “10) Canada”).
4. Choose **region** (e.g. “21) Pacific Time”).
5. `tzselect` suggests the `TZ` variable to export.

### 5.6 Changing System Time Zone (root)

1. **Backup** current zone file:

   ```bash
   cp /etc/localtime /etc/localtime.orig
   ```
2. **Link** desired zone:

   ```bash
   ln -sf /usr/share/zoneinfo/Australia/Sydney /etc/localtime
   ```
3. **Verify:**

   ```bash
   date
   ```

### 5.7 Setting the System Clock

* **Direct date format:**

  ```bash
  date MMDDhhmmYYYY.ss
  # e.g. date 032915262020.07   # Mar 29 15:26:07 2020
  ```
* **Human-readable:**

  ```bash
  date -s "Mon Mar 23 17:00:00 UTC 2020"
  ```
* **Viewing/formatting dates:**

  ```bash
  date --date="next Monday"
  date --date="1 year ago"
  date +"%Y-%m-%d %H:%M:%S"
  ```
