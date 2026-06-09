# Private testing page

The production website is hosted by GitHub Pages. GitHub Pages serves every
committed site file publicly and does not process `.htaccess`. Therefore, do
not commit `testing.html`, `testing.css`, or `.htaccess` to this repository.

## Safe deployment

Deploy the private testing page separately on an Apache server or another host
that supports server-side IP filtering. A suitable URL would be a dedicated
subdomain such as `testing.qazsociety.be`.

Before deploying:

1. Confirm the allowed user's current public IP using a trusted IP-checking
   service. Home and mobile public IP addresses may change.
2. Add that IP only to the private server's local `.htaccess`.
3. Keep a second authentication layer, such as HTTP Basic Authentication,
   because an IP allowlist alone is not reliable on shared networks.
4. Test from the allowed connection and from a different connection, such as
   mobile data. The second connection must receive HTTP `403 Forbidden`.
5. Confirm the private page does not appear in the public GitHub Pages build.

For a dedicated testing subdomain, place this `.htaccess` in the testing
document root so every file is protected:

```apache
Options -Indexes
Require ip 203.0.113.10
```

Replace `203.0.113.10` with the intended public IP. To allow an IPv6 address,
add another `Require ip` line.

If the testing page must remain inside the public website's document root,
limit the rule to its files instead. Every private asset must be included:

```apache
<FilesMatch "^(testing\.html|testing\.css)$">
    Require ip 203.0.113.10
</FilesMatch>
```

Do not rely on this rule when Apache is behind Cloudflare or another reverse
proxy until the server is configured to trust that proxy's client-IP header.
Otherwise Apache may see only the proxy's IP. Prefer the proxy provider's own
IP access rule, or correctly configure Apache `mod_remoteip`.

IP allowlisting is enforced only after deployment. Opening `testing.html`
directly from disk bypasses Apache and is useful only for local design checks.

## Local Git safeguards

The private files are listed in `.gitignore`. Before every commit, verify:

```sh
git status --short
git check-ignore -v .htaccess testing.html testing.css
git diff --cached --name-only
```

None of the private files should appear in `git diff --cached --name-only`.
Never use `git add -f` on them.

If `.htaccess` was previously tracked, adding it to `.gitignore` is not enough.
Remove it from Git's index without deleting the local file:

```sh
git rm --cached .htaccess
```

The resulting deletion must be committed once so future `.htaccess` changes
stay local. This does not erase copies from old Git history. If a real IP
address or credential was committed previously, treat it as exposed and rotate
or replace it.
