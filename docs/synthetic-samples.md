# Test Samples

Synthetic emails used to validate the detector. Each is designed to exercise specific detection signals so that individual code paths can be verified in isolation. Send any of these to the monitored inbox (or run manually) to reproduce results.

## Sample 1 — Multi-signal phishing (expected: `phishing`, high confidence)

```
Subject: Urgent: Your account will be suspended

Dear Customer, we detected unusual activity on your account.
You must verify immediately or your account will be suspended within 24 hours.
Click here to confirm your login: http://bca-verifikasi-secure.xyz/login
```

**Exercises:** brand impersonation (`bca` lookalike), risky TLD (`.xyz`), credential URL (`/login`), urgency keywords.
**Observed result:** `phishing`, 95% confidence, red flags: authentication failure, brand impersonation domain, credential-harvesting link, urgency pressure.

## Sample 2 — Legitimate personal mail (expected: `legitimate`, moderate confidence)

```
Subject: Greetings

Hi, just checking in. Let's catch up next week.
```

**Exercises:** no impersonation, no links, no urgency, auth failure only.
**Observed result:** `legitimate`, ~60% confidence (moderate due to unverified authentication — correct calibration).

## Sample 3 — Legitimate high-link newsletter (expected: `legitimate`, high confidence)

A real authenticated notification email (e.g. an Instagram "people to follow" digest) with 50+ links, all pointing to official brand/CDN domains.

**Exercises:** high URL count on legitimate mail (false-positive stress test), allowlisted brand CDNs.
**Observed result:** `legitimate`, 95% confidence — high link count correctly treated as normal.

## Additional single-signal samples (suggested)

- **URL shortener:** body containing a `bit.ly` link → tests `has_url_shortener`.
- **IP-address URL:** `http://192.0.2.10/login` → tests `has_ip_url`.
- **Display-name spoof:** From `"BCA Security" <random@gmail.com>` → tests `display_name_mismatch`.

## Testing note

Self-sent Gmail-to-Gmail test emails may not carry the DKIM/SPF/DMARC pass markers that real external mail includes, so legitimate self-tests often show as unverified. This is an artifact of the test method, not the detector. For authentic authentication behavior, test against real external mail or public phishing corpora.
