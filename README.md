# QWAC Trust Chain Verifier

> A browser-only tool that inspects real EU banks' TLS certificates through public Certificate Transparency logs and checks whether each issuer is an EU Qualified Trust Service Provider (QTSP) under eIDAS 2.0.

**Companion demo to my research paper:** *Security and Privacy Evaluation of Certificate Trust Chains in Digital Identity Wallet Onboarding for Banking: An Analysis Under the eIDAS 2.0 Architecture Reference Framework.*

**Key points**
- Fetches live data from **crt.sh**, the public Certificate Transparency log (RFC 9162)
- Cross-references certificate issuers against the **EU Trust List** of QTSPs
- Demo bank set is drawn from the **EBA Credit Institutions Register**
- Zero dependencies: one `index.html`, vanilla JS, no backend and no telemetry
- Clearly states what it does *not* do (no OID-level QWAC proof, no revocation checks), in line with the paper's threat model

A working demonstration that accompanies the research paper *Security and Privacy Evaluation of Certificate Trust Chains in Digital Identity Wallet Onboarding for Banking: An Analysis Under the eIDAS 2.0 Architecture Reference Framework*.

This is a single-file web application. There is no backend, no database, no telemetry. Open `index.html` in any modern browser and it runs.

---

## What this demonstrates

The research paper argues that the security and privacy of EU bank-wallet onboarding under eIDAS 2.0 depends on the QWAC certificate trust chain — and that this chain has measurable weaknesses (Section IV on STRIDE threats, Section V on privacy, Section VI on compliance gaps). This tool gives a live, hands-on demonstration of the first stage of that chain: looking at a real EU bank's domain, retrieving every TLS certificate ever issued for it from a public, append-only Certificate Transparency log, and identifying whether the issuing authority appears on the EU's official register of Qualified Trust Service Providers (QTSPs).

In one screen, the user can therefore see whether a given bank's public-facing infrastructure already relies on QWAC-class certificates (Section II.B of the paper), or whether it relies on standard commercial CAs (which connects directly to Compliance Gap 1 in Table IV — the lowest-common-denominator problem).

---

## The three authentic data sources

Every fact shown on the page traces back to one of three named, public, official sources. The page itself surfaces these in a side panel so the reader can verify any claim by clicking through.

### 1. Certificate Transparency — crt.sh

[`https://crt.sh`](https://crt.sh) is a public CT log search service operated by Sectigo. CT is defined by IETF RFC 9162 (referenced as [6] in the paper). Every TLS certificate from a CT-participating CA is logged in an append-only Merkle-tree log; once logged, an entry cannot be removed or altered. The verifier hits the crt.sh JSON endpoint directly from the browser:

```
https://crt.sh/?q=<domain>&output=json
```

The response is the raw list of CT log entries for that domain. The tool displays the crt.sh log ID for every certificate it shows, so your guide can click through and inspect the original log entry independently.

### 2. EU List of Trusted Lists — eidas.ec.europa.eu

[`https://eidas.ec.europa.eu/efda/tl-browser/`](https://eidas.ec.europa.eu/efda/tl-browser/) is the European Commission's official Trust List Browser. It is the authoritative register of every QTSP accredited by an EU member state's supervisory body. This is the same list referenced by ETSI EN 319 411-2 (paper reference [7]) and by Article 22 of eIDAS.

The verifier ships with a hard-coded subset of QTSPs that appear on this list. **This subset is not exhaustive and is not a substitute for the live Trust List.** The page tells the user to verify against the live list, and every "QTSP match" badge in the UI is paired with a direct link to the EU LOTL browser. The point is to demonstrate the cross-referencing pattern, not to replace the EU's own register.

### 3. EBA Credit Institutions Register — euclid.eba.europa.eu

[`https://euclid.eba.europa.eu/register/`](https://euclid.eba.europa.eu/register/) is the European Banking Authority's authoritative register of every authorised credit institution in the EU. This is the official answer to the question "which banks are in scope for eIDAS 2.0's wallet-acceptance mandate?" — because the mandate applies to authorised EU credit institutions.

The demo's pre-loaded list of domains is drawn from banks that appear on this register: Deutsche Bank (DE), BNP Paribas (FR), ING (NL), Santander (ES), UniCredit (IT), Intesa Sanpaolo (IT), BBVA (ES), Rabobank (NL). Each of these is a real, currently-licensed EU credit institution.

---

## Why these three sources and not others

A reasonable question you may ask is: why these sources and not, say, the bank's own website or a commercial security scanner. 
The answer connects to the paper's own methodology in Section VI.A:

> "Compliance gaps are identified by comparing (a) explicit requirements from eIDAS 2.0 [1] and ARF v1.4 [2], (b) ETSI standards operationalising those requirements [7]–[9], and (c) observed deployment practice from ENISA reports [14], [15], academic measurement studies [26], [27], and direct certificate inspection via publicly available tools (SSL Labs, crt.sh)."

crt.sh appears explicitly in the paper's methodology section as the certificate-inspection tool of choice. The EU Trust List Browser is the EU's own definition of which CAs count as Qualified. The EBA register is the EU's own definition of which entities count as banks under EU law. The chain of authority for every label and badge in the UI therefore terminates in an EU institution or in an IETF RFC.

---

## What the tool detects, and what it cannot

The verifier identifies a certificate as **likely QWAC** when the issuer Distinguished Name matches a known EU QTSP. This is a strong signal but not a definitive cryptographic proof, because the formal QWAC marker is the presence of the qualified-certificate policy OID `0.4.0.194121.1.4` in the Certificate Policies extension (paper Section II.B). That OID is embedded inside the certificate's DER bytes, and crt.sh's JSON endpoint does not always expose certificate extensions in a readily parseable form. Browsers, similarly, do not surface this OID through standard JavaScript APIs.

The tool is therefore transparent about this limitation. Every certificate card with a "QTSP" badge also links to its crt.sh page, where the full certificate can be downloaded and the OID confirmed manually. This honest scoping mirrors the paper's own distinction between *technical compliance* (a QWAC is nominally present) and *substantive compliance* (the full trust chain is correctly validated), drawn from Kosseff [24] and discussed in Section VI.A.

---

## How this connects to the paper, section by section

The demo is not a re-implementation of the paper's full framework — that would require backend infrastructure to perform OCSP queries, Trust List parsing, and audit logging, none of which is reasonable for a browser-only tool. What the demo does is illustrate, concretely, the *first stage* of the trust chain that the paper analyses.

When read alongside the paper:

The certificate listing on the right of each result is what the paper formalises as the certificate tuple in Equation (4), Section II.A. The "From EU QTSP" badge corresponds to membership of the trust anchor set *T* in Equation (5). The expiry calculation corresponds to the temporal predicate in Equation (6). The tool deliberately does not perform revocation checking, because the paper's central thesis is that revocation enforcement is broken in practice (Section IV, Threat T2; Section VI, Compliance Gap 3), and pretending otherwise in a demo would misrepresent the situation.

---

## Reproducibility

To reproduce any result shown by the tool, open the listed crt.sh URL for that certificate, or paste the domain into [crt.sh](https://crt.sh) directly. No data is hidden inside this application and no transformations are applied beyond deduplication of duplicate CT log entries for the same `(issuer, common_name, not_before)` tuple — a standard normalisation step because crt.sh returns one row per CT log, and most certificates are logged in multiple CT logs simultaneously.

---

## Honest scoping for the viva

The author wants to be clear about what this tool is and is not.

**It is** a faithful demonstration that real EU banks' certificate infrastructure can be inspected via Certificate Transparency, that the issuing authority can be cross-referenced against the EU's own register of QTSPs, and that this cross-reference produces a meaningful signal about whether the bank is using qualified versus standard TLS certificates. Every piece of information shown is fetched live from a public source named on-screen.

**It is not** a regulatory compliance scanner, a substitute for ETSI EN 319 411-2 conformity assessment, a real revocation checker, or a substitute for the EU's live Trust List. The paper's contribution is the analytical framework (STRIDE threat model, OCSP privacy model, eight-pillar mitigation framework, three evaluation matrices); the demo's contribution is to make the *entry point* of that framework tangible and inspectable.

---

## File contents

```
qwac-trust-verifier/
├── index.html      Self-contained app — open this in a browser
└── README.md       This file
```

No build step, no dependencies, no installation. The app uses only browser-native `fetch`, vanilla JavaScript, and three Google Fonts loaded over HTTPS.

---

## A note on what is *not* in this repository

The author has been explicit about one thing: this is a demonstration of the *first stage* of the trust chain (the certificate lookup and the issuer-to-QTSP mapping). A production-grade implementation of the paper's full framework would require, at minimum: an OCSP/OCSP-stapling validator, a Trust List parser that consumes the signed XML LOTL, an SD-JWT credential verifier, an mTLS handshake validator, and a structured audit log store. The eight-pillar mitigation framework in Table V of the paper describes the controls such a system would need to implement. Building it out is the natural next step beyond this demo.
