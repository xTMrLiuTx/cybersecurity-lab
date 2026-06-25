# Reports — Overview

This folder contains HTML reports and a short index describing where other report artifacts live in the repository.

Available reports (in this branch):

- reports/Nessus_1st_Report.html
  - Detailed Nessus vulnerability assessment rendered as a styled HTML report.
  - Open in a web browser to read (no special tooling required).

- reports/openVAS_Report.html
  - OpenVAS/Greenbone-style executive assessment rendered as HTML.
  - Open in a web browser.

Where the PDF lab reports live

- The PDF lab reports (Kali_Nmap_Lab_Report#2.pdf, Kali_mint_Scanners_Report#3.pdf, Kali_mint_Ubuntu_Report.pdf) are stored on the DOCS branch at the repository root. To download or review them:
  - Browse the DOCS branch on GitHub: https://github.com/xTMrLiuTx/cybersecurity-lab/tree/DOCS
  - Click the PDF file to view or download.

Why there are two locations

- Interactive/HTML reports are kept under `reports/` on the repository's default branch for easy viewing in a browser.
- Full, printable PDF lab reports are kept on the `DOCS` branch so the branch can be used to publish stable deliverables without mixing intermediate files into the main branch.

How to add or update reports

- Add a new HTML report: create or update `reports/<your_report>.html` on the default branch (main).
- Add new PDFs: commit them to the `DOCS` branch (this keeps deliverables separate).
- When adding or removing files, please update this README with the filename and a one-line description.

Quick notes

- If you'd like I can also move the PDFs into `reports/` on the default branch (or create a `reports/pdfs/` folder) — tell me which behaviour you prefer.
