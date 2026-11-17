## Title
Northbridge Institute — Directory Listing Enabled on /assets Exposing Embargoed Research & Confidential Pre-Publication Documents

## Description
**Affected URL:** https://researcher.pwnedbox.site/
The Northbridge Institute research portal has directory listing enabled on the /assets path, allowing any unauthenticated user to browse and download all files stored there. The exposed files include an unpublished 2025 research report, a conference draft, a grant budget allocation spreadsheet, an internal review document, and critically — a peer review embargo schedule, which reveals which papers are under embargo and their planned publication timeline. All files are freely downloadable with no authentication required.

## Steps to Reproduce
1. Navigate to `https://researcher.pwnedbox.site/assets` in a browser
2. Observe the server returns a full directory index with no authentication prompt
3. Click any file to download it directly — no login or session required

## POC (Proof of Concept)
<img width="1918" height="997" alt="f" src="https://github.com/user-attachments/assets/a3f221ed-66fa-49ac-b697-7abf76e138ec" />

## Impact
- `peer_review_embargo_schedule.txt` exposed → reveals which papers are under embargo and planned publication dates → academic integrity violation, competitors can scoop unpublished research
- `conference_draft_pdf.pdf` & `2025_research_report_pdf.pdf` exposed → pre-publication research accessible before authors/journals have approved release
- `grant_budget_allocation_2026.csv` exposed → confidential funding figures and budget plans disclosed
- `internal_review_pdf.pdf` exposed → internal peer critique and review notes accessible to public


## Remediation
- Disable directory listing: `Options -Indexes` (Apache) or `autoindex off` (Nginx)
- Move `/assets` outside the web root; serve files only via authenticated server-side logic with role checks
- Apply embargo-aware access control — embargoed documents must only be accessible to authorized researchers until the release date
- Audit all other paths on `researcher.pwnedbox.site` for similar misconfigurations
