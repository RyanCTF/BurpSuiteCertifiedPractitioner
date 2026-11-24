# BSCP Exam - Strategy Notes

## Stage-to-vuln map (focus the search per stage)
The base blog app is identical in every lab/exam instance; at each step, diff the NEW functionality
that just became reachable and test only the stage-appropriate vuln classes against it.

| Vuln class | Stage 1 | Stage 2 | Stage 3 |
|---|---|---|---|
| SQL injection | | yes | yes |
| XSS / DOM-based / CSRF / clickjacking / CORS | yes | yes | |
| Access control / Authentication / JWT / OAuth | yes | yes | |
| HTTP request smuggling / Host-header / Web cache poisoning | yes | yes | |
| XXE | | | yes |
| SSRF | | | yes |
| OS command injection | | | yes |
| SSTI | | | yes |
| Directory traversal | | | yes |
| Insecure deserialization | | | yes |
| File upload | | | yes |

The 7 file-read classes (XXE, SSRF, OS cmd, SSTI, directory traversal, insecure deserialization,
file upload) are **stage-3 only** - they read `/home/carlos/secret`. Stages 1 and 2 are get-a-
foothold then escalate-to-admin, never a file-read class. Don't hunt command injection at the start.

Note: SQLi spans stages - it's the usual stage-2 privesc, and it can ALSO be the stage-3 file read when the DB account has file privileges (`load_file`/`COPY`/`pg_read_file` to read `/home/carlos/secret`).

Exam objective recap: Stage 1 = low-privileged user account, Stage 2 = administrator account,
Stage 3 = read `/home/carlos/secret` and submit its content.

## Strategy
1. **Identify the new functionality.** Same base site each step, so after you gain access, diff what
   is now exposed that was not reachable before - that delta is where the next-stage bug lives.
2. **Selective scanning, started early.** Automated scans are slow; kick them off the moment you
   have an input, and focus them on specific inputs (PortSwigger's augmenting-manual-testing-with-
   burp-scanner guide). Prebuild scan configs per vuln class to speed this up.
3. **Reuse lab-solution PoC.** You will want working exploit code; copy it from the matching Academy
   lab solution and adapt. There are few, if any, curveballs outside the labs.
4. **Don't tunnel-vision.** If a feature resists one vuln class, step back and try another; the
   wording of pages/instructions is often a hint to the intended bug or what's happening behind
   the scenes.
5. **Match the vuln to the stage** (see table) - saves time vs. testing every class everywhere.
6. **Stage-1 creds hint.** The instructions give a username list and a password list - that is a
   nudge toward username enumeration + password spraying. Read all wording; it is there for a reason.

## Logistics
7. Take the practice exam first to get familiar with the format.
8. Register the proctoring site with the exact same email as your PortSwigger account - the
   proctoring flow may not recognize the exam otherwise.
9. After passing, check the PortSwigger site for results - you may not get an email; results and
   proctoring verification take ~24-48h and appear on the site after the full 4-hour window.
