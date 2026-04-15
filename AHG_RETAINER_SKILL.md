# AHG Retainer Form Auto-Fill Tool — Project Skill

## Project overview

A desktop application for Law Office of Andrew H. Griffin, III, APC that auto-fills the firm's Chapter 7 Bankruptcy Retainer Agreement from a simple input form. Andrew inputs client details, clicks generate, and receives a fully populated Word document (.docx) ready to print and sign.

This is NOT a web app. It is a native desktop application that installs and runs like standard Windows software — no browser, no internet required (except optional Claude API features), no accounts, no hosting.

---

## Client context

- Law Office of Andrew H. Griffin, III, APC
- El Cajon, CA — solo bankruptcy attorney
- Primary use: Chapter 7 bankruptcy retainer agreements
- Pain point: manually filling in repetitive client info across a multi-page legal contract
- Technical sophistication: low — must feel like normal installed software, zero learning curve
- Built and maintained by KKAI (Kevin Kakkary Automation Services LLC)

---

## Tech stack

- **Shell:** Electron — packages the app as a native Windows .exe installer
- **UI:** React inside Electron — familiar stack, no browser chrome visible to user
- **Document generation:** docx npm library — generates a properly formatted .docx file
- **Styling:** Tailwind CSS — clean, professional, law office appropriate
- **Packaging:** electron-builder — produces a Windows .exe installer Andrew runs once

No backend server. No hosting. No cloud. Everything runs locally on Andrew's machine.

---

## Retainer form fields

Extracted from the actual Chapter 7 Bankruptcy Retainer Agreement template.

### Dynamic fields (user inputs these)

| Field | Description |
|---|---|
| `contract_date` | Date of the contract (e.g. March 27, 2025) |
| `client_name` | Primary debtor full legal name |
| `co_debtor_name` | Co-debtor full legal name (optional — joint filing) |
| `attorney_fee` | Total attorney fee (default $2,433.00) |
| `discounted_fee` | Discounted fee if paid at signing (default $2,333.00) |
| `debtor_address` | Primary debtor street address |
| `debtor_city_state_zip` | City, state, zip |
| `debtor_phone` | Debtor phone number |
| `debtor_email` | Debtor email address |
| `co_debtor_address` | Co-debtor address (optional) |
| `co_debtor_city_state_zip` | Co-debtor city, state, zip (optional) |
| `co_debtor_phone` | Co-debtor phone (optional) |
| `co_debtor_email` | Co-debtor email (optional) |
| `debtor_signed_date` | Date debtor signs |
| `co_debtor_signed_date` | Date co-debtor signs (optional) |
| `retainer_paid` | Amount of retainer paid at signing |

### Static fields (hardcoded — never change)

- Attorney name: Andrew H. Griffin, III
- Firm name: Law Office of Andrew H. Griffin, III, APC
- Address: 275 E. Douglas Avenue, Suite 112, El Cajon, California 92020
- Phone: 619.440.5000 / Fax: 619.440.5991
- Email: andrew@andrewgriffinlawoffice.com
- Standard attorney fee: $2,433.00
- Discounted fee: $2,333.00
- Hourly rate for additional services: $495.00/hour
- Chapter 7 filing fee: $338.00
- Chapter 13 filing fee: $313.00
- Returned check fee: $55.00
- All numbered service clauses (Sections I, II, III, IV) — boilerplate, never change
- All statutory disclosure language — boilerplate, never change

---

## UI layout

Single window, clean two-column layout:

**Left panel — input form:**
- Section: Client information
  - Contract date (date picker, defaults to today)
  - Client name (text input)
  - Is this a joint filing? (toggle — reveals co-debtor fields)
  - Co-debtor name (conditional)
- Section: Contact information
  - Debtor address, city/state/zip, phone, email
  - Co-debtor address, phone, email (conditional, shown if joint filing)
- Section: Fees
  - Attorney fee (pre-filled $2,433.00, editable)
  - Discounted fee (pre-filled $2,333.00, editable)
  - Retainer paid at signing
  - Payment plan? (toggle — reveals payment plan note)
- Section: Signatures
  - Debtor signed date
  - Co-debtor signed date (conditional)

**Right panel — live preview:**
- Shows key populated fields in a clean summary card
- Client name, date, fee, joint filing status
- Not a full document preview — just a confirmation of what will be generated

**Bottom bar:**
- "Generate Retainer" button — prominent, primary action
- Output goes directly to a user-selected folder or Documents by default
- Success message: "Retainer saved to [path]"

---

## Document generation

Use the `docx` npm library to generate the .docx file programmatically.

### Key implementation notes

- Page size: US Letter (12240 x 15840 DXA), 1 inch margins
- Font: Times New Roman or Arial — match the original template
- Preserve all original formatting: centered headings, justified body text, bold section headers
- Signature lines: use tab stops to create the blank lines, not tables
- Page numbers: bottom center, format "Page N of N"
- Client initials footer: bottom of each page shows "______Client ______Client" as in original
- The firm letterhead (logo + address block) appears at the top of page 1 only

### Template population approach

Build the full document structure in JavaScript using the docx library. Every static clause is hardcoded as a Paragraph. Dynamic fields are injected as TextRun values from the form inputs.

```javascript
// Example: contract header
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [
    new TextRun({ text: contractDate, bold: false }),
  ]
}),
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [
    new TextRun({
      text: `Contract between Law Office of Andrew H. Griffin, III, APC (a "Debt Relief Agency")`,
      bold: true
    }),
  ]
}),
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [
    new TextRun({ text: `${clientName} ("Client(s)")`, bold: true }),
  ]
}),
```

### Output

- File saved as: `Griffin_Retainer_[ClientLastName]_[Date].docx`
- Saved to: user's Documents folder by default, or folder they choose via save dialog
- Format: .docx (opens in Word, Google Docs, LibreOffice)

---

## Document structure (section order)

Reproduce exactly in this order:

1. Firm letterhead (page 1 only)
2. Contract date and party names (centered, bold)
3. Opening paragraph (engagement agreement language)
4. **Section I** — Services Included in the Initial Fee (numbered list 1-12)
5. **Fees and Charges** — attorney fee, discounted fee, payment terms, hourly rate
6. **Section II** — Additional Services Subject to Additional Fee (numbered list 1-9)
7. **Section III** — Additional Services Not Included (numbered list 1-10)
8. **Section IV** — Duties and Responsibilities of the Debtor (numbered list 1-17)
9. **Acknowledgement of Receipt of Disclosures**
10. Statutory notices (§342(b), §527(a), §527(b), §527(c))
11. Debt relief agency disclosure (bold caps)
12. Signature block — attorney, debtor, co-debtor
13. Notice to Clients Under §527(b)(2) (numbered list 1-7)
14. Terms and Definitions Addendum header
15. Instructions on Providing Information (initialed list 1-9)
16. Final signature block with dates, addresses, phones, emails
17. Retainer paid line
18. Returned check notice

---

## Electron packaging

Use `electron-builder` to package as a Windows installer (.exe).

```json
// package.json build config
{
  "build": {
    "appId": "com.kkai.ahg-retainer",
    "productName": "AHG Retainer Tool",
    "win": {
      "target": "nsis",
      "icon": "assets/icon.ico"
    },
    "nsis": {
      "oneClick": true,
      "perMachine": false
    }
  }
}
```

Andrew runs the installer once, app appears in Start Menu as "AHG Retainer Tool".

---

## Optional Claude API enhancement (Phase 2)

Once the base form-fill tool is working, optionally add a Claude API call that:
- Takes minimal input (client name, basic situation)
- Pre-fills suggested fee amounts based on case complexity
- Flags if any fields look inconsistent before generating

This is optional and not required for v1. v1 is purely form → document generation, no AI required.

---

## Key decisions log

- **Electron over web app:** Andrew wants software that feels installed and native, not a website
- **docx library over PDF:** Word doc is easier for Andrew to make last-minute edits before printing
- **No backend, no cloud:** Everything local — no hosting costs, no internet dependency, no privacy concerns about client data leaving the machine
- **Static clauses hardcoded:** All boilerplate legal language is hardcoded in the document generator, not stored in a database — simpler and more reliable
- **Joint filing toggle:** Co-debtor fields only appear when needed — keeps the form clean for single filers
- **Default fees pre-filled:** $2,433.00 and $2,333.00 are pre-filled but editable in case fees change
- **Output as .docx not PDF:** Andrew may want to make edits before printing; Word format preserves that flexibility
- **Phase 2 Claude API:** AI enhancement is optional and deferred — v1 ships as a pure form tool

---

## What this is NOT

- Not a web app or anything that opens in a browser
- Not a document review tool (see AHG_SKILL.md for that separate product)
- Not a case management system
- Not connected to MyCase or any external service
- Not storing any client data — each retainer is generated and saved, nothing persisted in the app
