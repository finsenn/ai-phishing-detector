# Detection Signals Reference

Every signal below is computed deterministically in the feature-extraction node and passed to the LLM for reasoning. No signal alone produces a verdict — the model weighs them in combination.

| Signal | What it detects | Why it matters |
|---|---|---|
| `sender_impersonation` | Sender domain contains a known brand name but is not that brand's official domain | Classic typosquatting, e.g. `bca-verifikasi.xyz` impersonating BCA |
| `link_impersonations` | A link domain impersonates a brand (same logic applied to URLs in the body) | Phishing links frequently use lookalike domains that pass authentication |
| `reply_to_mismatch` | The Reply-To domain differs from the From domain | Attackers route replies to a domain they control |
| `display_name_mismatch` | Display name claims a brand the actual sending domain doesn't match | The "PayPal Security &lt;random@gmail.com&gt;" trick |
| `has_url_shortener` | A link uses a shortener (bit.ly, tinyurl, t.co, etc.) | Hides the true destination from the reader and from inspection |
| `has_ip_url` | A link points to a raw IP address instead of a domain | Legitimate organizations use domains; raw IPs signal evasion |
| `credential_url_hits` | Count of links containing login / verify / secure / account / password | Indicates credential-harvesting landing pages |
| `has_risky_tld` | A link uses a TLD over-represented in abuse (.xyz, .top, .tk, .ml, .icu, .click, .gq, .cf) | Cheap or free TLDs are favored by attackers |
| `urgency_keywords` | Pressure phrases (urgent, suspended, within 24 hours, account locked, etc.) | Manufactured urgency is designed to bypass careful judgment |
| `dkim_pass` / `spf_pass` / `dmarc_pass` | Email authentication results | Failing all three *combined with* other signals raises risk; failing alone is weak evidence |
| `url_count` | Total number of links in the email | High counts are normal for newsletters; only meaningful alongside other signals |
| `unique_link_domains` | Deduplicated list of link domains | Basis for lookalike analysis and sender/link consistency checks |

## How the LLM uses these

The system prompt instructs the model that:

- Authentication passing does **not** guarantee safety (attacker-owned lookalikes pass auth).
- A high URL count is **normal** for legitimate notifications and is not suspicious on its own.
- Authentication failure **alone** is weak evidence — personal mail often fails.
- The genuinely dangerous pattern is a **claimed trusted identity combined with** lookalike/mismatched domains, auth failures, credential-harvesting links, and urgency.
- The impersonation flags are **heuristic hints**, not proof, and may be overridden when all other signals indicate a recognized legitimate service.
