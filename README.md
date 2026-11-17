Log Analyser

A Python tool that reads an authentication log and flags suspicious sign-in activity, the kind of thing a security analyst or IT support technician checks when an account might be under attack.

Everything is in one file, `log_analyser.py`: the analyser, a sample data generator and the unit tests. It uses only the Python standard library, so there is nothing to install.

Why I built it

I am building my skills towards a career in cybersecurity and data. I wanted a project that showed how raw log data can be turned into clear, useful findings, so I built a tool that spots brute force attacks, flags a possible account compromise and highlights unusual login times.

 What it detects

| Severity | Check | What it means |
|----------|-------|---------------|
| CRITICAL | Possible compromise | A successful login from an IP address straight after it made a burst of failed attempts |
| HIGH | Brute force | Many failed logins from one IP address within a short time window |
| MEDIUM | Off-hours login | A successful login outside 06:00 to 22:00 |

It also prints a summary (total logins, top IP addresses by failed attempts, most targeted accounts) and saves every finding to a CSV file that opens in Excel.

Getting started

You need Python 3.9 or newer.

```bash
# Run the unit tests
python log_analyser.py --test

# Create a made-up sample log and analyse it
python log_analyser.py --demo

# Analyse your own log file
python log_analyser.py yourfile.log

# Change the sensitivity
python log_analyser.py yourfile.log --threshold 3 --window 5
```

On Mac or Linux, use `python3` instead of `python`.

Options

| Option | Description | Default |
|--------|-------------|---------|
| `--demo` | Create a sample log and analyse it | off |
| `--test` | Run the unit tests | off |
| `--threshold` | Failed logins that count as brute force | 5 |
| `--window` | Time window in minutes | 10 |
| `--output` | Where to save the CSV report | `reports/findings.csv` |

Log format

One event per line:

```
2026-11-10 03:12:44 LOGIN_FAILED user=admin ip=203.0.113.45
2026-11-10 03:18:00 LOGIN_SUCCESS user=admin ip=203.0.113.45
``

Lines that do not match this format are counted and skipped, so one bad line does not crash the program. All IP addresses in the sample data come from ranges reserved for documentation, so no real data is involved.

## Example output

```
FINDINGS (3)
  [CRITICAL] Possible compromise: 203.0.113.45 (user: admin) at 2026-11-10 03:18:00
      Successful login after a brute force burst. Reset the password and review the account.
  [HIGH] Brute force: 203.0.113.45 (user: admin, root) at 2026-11-10 03:13:20
      Up to 14 failed logins within 10 minutes, targeting 2 account(s)
  [MEDIUM] Off-hours login: 198.51.100.10 (user: dev) at 2026-11-10 23:41:00
      Successful login outside 06:00 to 22:00
```

The CSV report contains the same findings, so they can be filtered and sorted in Excel.

<!-- Add a screenshot of your terminal output or your CSV opened in Excel here:
![Example output](screenshot.png) -->

How it works

1. **Parse:** regular expressions turn each log line into a structured record.
2. **Detect brute force:** failed logins are grouped by IP address, then a sliding time window counts attempts. This is more reliable than a simple total, because a few failures spread over a day are normal, but many within minutes are not.
3. **Link events:** a successful login from an IP that was already flagged is escalated to CRITICAL.
4. **Report:** findings are sorted by severity, printed to the screen and saved to CSV.

Tests

The built-in tests cover:

- A burst of failed logins is flagged as brute force
- Slow, spread-out failures are not flagged
- The peak number of attempts is reported correctly
- Night-time logins are flagged and daytime logins are not
- A successful login after a burst is escalated to critical
- A normal login produces no findings

What I learned

- Writing regular expressions to parse structured text
- Using a sliding window to detect patterns over time
- Handling bad input safely instead of crashing
- Writing unit tests for detection logic, including cases that should not trigger
- Turning technical results into a clear, prioritised report

Ideas for extending it

- Read real log formats such as Linux `auth.log` or Windows Event Logs
- Add a chart of failed logins per hour
- Compare IP addresses against a list of known bad addresses
- Send an email alert when a CRITICAL finding appears

Skills shown

Python, data parsing, regular expressions, security monitoring concepts, unit testing, command line tools and clear documentation.
