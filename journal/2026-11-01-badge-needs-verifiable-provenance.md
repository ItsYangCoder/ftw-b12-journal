# Journal - 2026-11-01 - A Badge Image Still Needs Verifiable Provenance

## Today in one sentence
- Preparing my Databricks Data Engineer Associate badge for sharing reminded me that a badge image is the presentation layer, while trust comes from structured metadata, issuer identity, evidence, status, and integrity checks.

## What I learned
- A digital badge has two very different forms: the **visual representation** and the **credential record**. The image is useful because people can recognize the achievement quickly. However, pixels alone do not prove who issued the credential, who earned it, when it was issued, what criteria were satisfied, or whether it is still valid.
- This is similar to a Data Engineering dashboard. A chart can communicate a result, but it is not the source table, transformation logic, test history, or lineage. If someone copies the chart into another file, the numbers still look convincing even though the connection to the governed source is gone. In the same way, a copied or regenerated badge image can look correct without being independently verifiable.
- A trustworthy credential needs **provenance**: information about where the claim came from and how it can be checked. The W3C Verifiable Credentials model describes an issuer, a credential subject, claims about that subject, validity dates, status, schemas, and optional evidence. A verifier can use the credential's securing mechanism to check issuer authenticity and whether the data was changed after issuance.
- The roles are simple. The **issuer** makes the claim, the **holder** receives and presents it, and the **verifier** decides whether to trust it. This maps neatly to a pipeline: a source publishes records, a platform carries them, and a downstream consumer validates the source, schema, freshness, and integrity before using them.
- Evidence and integrity are not the same thing. Evidence explains why an achievement was awarded—for example, completion criteria or an assessment result. A cryptographic proof or signature helps detect tampering and authenticate the issuer. A file can contain strong evidence but still lack a trustworthy signature, or it can be correctly signed while making a claim that a verifier does not consider relevant.
- **Credential status** matters because data can change after issuance. A credential might expire, be revoked, or be superseded. This is comparable to a slowly changing dimension or a data contract version: keeping one old snapshot does not guarantee that it still represents the current state.
- A verification URL is more valuable than a screenshot because it lets a consumer retrieve current metadata from the issuing system. However, the URL itself should be checked carefully. A convincing page on an unrelated domain is not the same as a record controlled by the issuer or its authorized credential platform.
- Hashes and digital signatures solve different problems. A hash can show that two files have the same bytes or that a referenced artifact has not changed. A signature connects that integrity check to a key controlled by an identified issuer. In pipeline terms, a checksum can detect file changes, but it does not automatically tell me whether the source was authorized or whether the business meaning is correct.
- Standards improve interoperability by giving systems consistent field names and expected structures. Open Badges 3.0 aligns achievement credentials with the W3C Verifiable Credentials model so wallets and verification tools can process issuer, learner, achievement, criteria, evidence, and proof information consistently. I am using that standard as a learning model; I am not claiming that this specific PNG file contains an Open Badges 3.0 credential.
- Privacy is part of credential sharing. A public journal does not need my candidate ID, exam registration details, score report, email address, or private credential-management URL. The clean badge graphic communicates the topic without exposing those fields. If verification is needed for an application, the official verification record should be shared through the appropriate channel instead of publishing personal exam documents everywhere.
- The same principle applies to project evidence. A green CI screenshot, a dashboard image, or a successful job page is helpful, but the stronger evidence is a traceable chain: repository revision, workflow run, environment, tests, data inputs, output tables, and current status. Presentation should summarize the evidence, not replace it.

## Terms I am still learning
- **Digital credential** - a structured record containing claims about an achievement and the information needed to interpret or verify those claims.
- **Provenance** - the origin and history of data, including who created it, where it came from, and how it changed.
- **Issuer** - the organization or authority that makes and signs a credential claim.
- **Credential subject** - the person or entity that the credential's claims describe.
- **Verifier** - the party that checks the credential and decides whether it is trustworthy and relevant.
- **Evidence** - supporting information showing why the credential was issued or why a claim should be believed.
- **Integrity** - confidence that the credential data has not been altered without detection.
- **Credential status** - information indicating whether a credential remains active, has expired, or has been revoked or superseded.
- **Interoperability** - the ability of different systems to exchange and interpret the same credential consistently.

## What confused me
- I initially treated the badge image as the credential itself because the visual is what I see and share. The image is really only one representation. A verifier needs the issuer-controlled record or a properly secured credential payload to validate the claim.
- I also had to separate “the file has not changed” from “the claim is trustworthy.” A checksum can prove byte-level consistency, but it cannot prove that the issuer is legitimate, the achievement criteria were appropriate, or the credential is still active.
- I still want to understand which credential format and verification platform Databricks currently uses for this certification. I do not have enough verified metadata from the image alone to state whether the issued credential is specifically an Open Badges 3.0 or W3C Verifiable Credential implementation.

## One small next step
- [ ] Open the official credential portal and record the issuer, issue date, credential name, current status, and public verification URL in a private career tracker; keep candidate IDs, exam details, and account links out of the public journal.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will treat the clean badge graphic as an illustration of the achievement, not as independent proof of the credential.
- I will use standards language such as issuer, subject, evidence, integrity, and status as a conceptual model without claiming that the specific Databricks badge PNG implements a particular credential standard.
- I did not claim a new pipeline implementation today. The latest visible PondoLens repository activity remains October 22, so this is a learning reflection grounded in today's certification-badge work.
- I excluded the phone screenshots because the badge-only asset communicates the topic with less personal and device information.

## Evidence from today (optional)
- [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model/) - defines issuer, credential subject, validity, status, evidence, and securing mechanisms.
- [1EdTech Open Badges 3.0 specification](https://standards.1edtech.org/open-badges/specifications/standards/v3p0) - describes portable achievement credentials aligned with the Verifiable Credentials model.
- [Latest visible PondoLens commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/d684dff932dea289a8435fa2d1666990797e500f) - October 22 evidence; no newer project commit was presented as today's work.

![Databricks Certified Data Engineer Associate badge graphic on a transparent background](../assets/2026-11-01-databricks-data-engineer-badge.webp)

This inspected badge-only graphic contains no name, credential ID, account details, private URL, or exam record. It illustrates the achievement while the journal explains why an image should not be confused with the issuer's verifiable credential record.

## Reflection (optional)
- What felt easy today? The difference between a badge image and a credential record became clear when I compared it with a dashboard versus its source tables and lineage.
- What felt difficult today? It is tempting to treat something that looks official as sufficient evidence. Trust requires more work: verifying the issuer, current status, integrity, and context of the claim.
- What do I want to understand better next time? I want to inspect an actual machine-readable credential payload and trace how a verifier checks its signature, schema, status, and issuer identity.

## Mood or meme (optional)
- A badge is the dashboard tile; provenance is the pipeline behind it.
