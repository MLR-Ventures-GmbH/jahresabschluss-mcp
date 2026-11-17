# Tools

Snapshot of the server's tool catalogue on 17 November 2026: 125 tools (55 with the right `read`, 65 with `write`, 3 with `send`, 1 with `finalize`, 1 with `file`).

The live list is what `tools/list` returns for your connection: a client only sees the tools its rights and the company's modules allow. Tool names are stable identifiers; titles, descriptions and several response fields are German, as returned by the server. The English column is a short description for this page.

## Rights

| Right | Meaning | Granted by |
| --- | --- | --- |
| `read` | All reading tools, including exports. Part of every connection. | Sign-in (OAuth) and API key |
| `write` | Post and reverse entries, issue invoices, record payments, book receipts and bank transactions, maintain master data, check and prepare tax filings. | Sign-in (OAuth) and API key |
| `send` | E-mail invoices, quotes and payment reminders, only to addresses stored for the business partner or noted on the invoice. | Sign-in (OAuth) and API key |
| `finalize` | Lock periods irrevocably (Festschreibung under GoBD). | API key only |
| `file` | Transmit to the tax office. Not granted at present; filing happens in the app through the approval link. | not granted |

An API key can also carry the right `webhooks`; webhooks are managed through the REST API, not through MCP.

## Annotations

Every tool carries the MCP tool annotations `readOnlyHint`, `destructiveHint`, `idempotentHint` and `openWorldHint`. The column "Annotations" lists the ones that are set:

- **read-only**: the tool does not change anything.
- **destructive**: the effect cannot be taken back, for example an issued invoice number, a deleted receipt, a locked period or a sent e-mail.
- **idempotent**: repeating the same call does not create a second result. Tools that post entries, invoices or payments take an idempotency key for this; a retry after a timeout reuses the same key and the same data.
- **open-world**: the tool reaches third parties (e-mail to a business partner, transmission to the tax office).

## Which tools a connection sees

The rights of the connection decide first. The modules the company uses then narrow the list:

- Bookkeeping or EÜR (cash-basis) module: all tools.
- Tax module only: `list_tax_deadlines`, `list_tax_filings`, `check_tax_filing`, `create_tax_draft`, `prepare_tax_filing`, `get_tax_operation`, `request_tax_filing_approval`, `submit_tax_filing`, `get_vat_supplementary`, `confirm_vat_supplementary`, `get_entity`, `list_fiscal_years`, `get_vat_return`.
- Annual financial statements only: `get_entity`, `list_fiscal_years`, `upload_trial_balance`, `get_annual_report_status`, `list_annual_reports`.

## Company, accounts and ledger

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `get_entity` | `read` | read-only, idempotent | Stammdaten des Unternehmens | Company master data: name, legal form, size class, chart of accounts, small-business status, VAT accounting method and profit determination (balance sheet or EÜR); also says whether the connection belongs to a sandbox. |
| `get_entity_settings` | `read` | read-only, idempotent | Einstellungen der Gesellschaft | Tax and bookkeeping settings: chart of accounts, effective profit determination, small-business status, VAT method, VAT return period, permanent extension, balance sheet date and whether tax numbers and VAT ID are on file. Settings are changed in the app. |
| `list_fiscal_years` | `read` | read-only, idempotent | Wirtschaftsjahre | All fiscal years with year, period and status. |
| `create_fiscal_year` | `write` | idempotent | Geschäftsjahr anlegen | Creates the current fiscal year, or the one ending in a given year (at most the following year). |
| `list_accounts` | `read` | read-only, idempotent | Kontenrahmen | Chart of accounts (SKR03 or SKR04) including the company's own accounts, with an optional text filter. |
| `list_hgb_positions` | `read` | read-only, idempotent | Posten in Bilanz und GuV | The balance sheet and profit-and-loss line items an account can be assigned to. |
| `upsert_account` | `write` | idempotent | Konto anlegen oder ändern | Renames or hides a catalogue account, assigns it to another balance sheet or profit-and-loss item, or creates an own ledger account. |
| `list_journal_entries` | `read` | read-only, idempotent | Buchungsjournal | Journal in ascending entry number, filterable by fiscal year, date range, account, origin and document number, or only entries after a given number. |
| `get_journal_entry` | `read` | read-only, idempotent | Einzelne Buchung | One journal entry by ID. |
| `create_journal_entry` | `write` | idempotent | Buchung anlegen | Posts a manual journal entry (debit account, credit account, amount, tax key) through the booking core. |
| `storno_journal_entry` | `write` | idempotent | Buchung stornieren | Reverses a journal entry by counter-entry (reversal instead of deletion). |
| `import_journal_batch` | `write` | idempotent | Buchungsstapel importieren | Posts already balanced journal lines as one batch (up to 5,000 lines, all or nothing); identical lines of an earlier import are skipped; supports a dry run. |
| `list_journal_batches` | `read` | read-only, idempotent | Buchungsstapel | Journal imports of the company with their batch ID. |
| `storno_journal_batch` | `write` | destructive, idempotent | Buchungsimport stornieren | Reverses all not yet reversed entries of an import in one transaction (all or nothing). |
| `get_saldenliste` | `read` | read-only, idempotent | Summen- und Saldenliste | Trial balance (Summen- und Saldenliste) of a fiscal year, optionally as of a date within the year. |
| `get_opening_balances` | `read` | read-only, idempotent | Stand der Eröffnungswerte | Whether a fiscal year's opening balances are posted, where they come from, whether the previous year is final, and deviations from the previous year's closing balance sheet. |
| `carry_forward_opening_balances` | `write` | idempotent | Eröffnungswerte vortragen | Carries the closing balance sheet of the final previous year forward as opening balances; nothing is posted from a previous year that is still open. |
| `export_bookkeeping` | `read` | read-only, idempotent | Export eines Geschäftsjahres | Export of a fiscal year: journal and trial balance as CSV text; for posting batches in DATEV format and the GoBD package it names the download path instead of passing the file through the model. |

## Business partners

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_business_partners` | `read` | read-only, idempotent | Geschäftspartner | Customers and suppliers with master data (address, VAT ID, IBAN, e-mail, payment terms, sub-ledger account). |
| `get_business_partner` | `read` | read-only, idempotent | Geschäftspartner mit Historie | One business partner with its change history. |
| `create_business_partner` | `write` | idempotent | Geschäftspartner anlegen | Creates a customer or supplier without bank details (those are maintained in the app); e-mail addresses additionally need the right `send`. |
| `update_business_partner` | `write` | idempotent | Geschäftspartner ändern | Changes the given fields of a business partner; every change is recorded in the master-data history; IBAN and BIC cannot be changed here. |

## Sales invoices, drafts and recurring invoices

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_outgoing_invoices` | `read` | read-only, idempotent | Ausgangsrechnungen | Sales invoices, filterable by fiscal year and status. |
| `get_outgoing_invoice_document` | `read` | read-only, idempotent | Rechnungsdokument abrufen | The file of a sales invoice as PDF, ZUGFeRD or XRechnung (base64), with the result of the e-invoice validation. |
| `create_outgoing_invoice` | `write` | destructive, idempotent | Ausgangsrechnung ausstellen | Issues a sales invoice and posts revenue, VAT and receivable. The invoice gets its number and can afterwards only be cancelled. |
| `send_outgoing_invoice` | `send` | destructive, idempotent, open-world | Ausgangsrechnung per E-Mail senden | E-mails an issued invoice as PDF, ZUGFeRD or XRechnung, only to an address stored for the business partner or noted on the invoice; at most 3 dispatches per invoice and 100 per company in 24 hours. |
| `cancel_outgoing_invoice` | `write` | destructive, idempotent | Ausgangsrechnung stornieren | Issues a cancellation invoice for a sales invoice (own number, counter-entry of revenue and VAT, open item closed). |
| `create_outgoing_credit_note` | `write` | destructive, idempotent | Gutschrift zu Ausgangsrechnung ausstellen | Issues a credit note (partial cancellation) for a sales invoice, with its own number and a proportional counter-entry of revenue and VAT. |
| `list_invoice_drafts` | `read` | read-only, idempotent | Rechnungsentwürfe lesen | Invoice drafts without invoice number and without posting, filterable by status. |
| `get_invoice_draft` | `read` | read-only, idempotent | Rechnungsentwurf lesen | One invoice draft with its line items. |
| `create_invoice_draft` | `write` | idempotent | Rechnungsentwurf anlegen | Creates a draft: no invoice number, no posting, no dispatch. |
| `update_invoice_draft` | `write` | idempotent | Rechnungsentwurf ändern | Changes the given fields of an open draft and detects a change made in the meantime. |
| `issue_invoice_draft` | `write` | destructive, idempotent | Rechnungsentwurf ausstellen | Issues a draft: assigns the consecutive invoice number and posts revenue, VAT and receivable; reversible only through a cancellation invoice. |
| `discard_invoice_draft` | `write` | destructive, idempotent | Rechnungsentwurf verwerfen | Discards an open draft as long as no invoice exists; leaves no gap in the number sequence. |
| `list_invoice_subscriptions` | `read` | read-only, idempotent | Rechnungsabos lesen | Recurring invoices with line items, interval and next date. |
| `get_invoice_subscription` | `read` | read-only, idempotent | Rechnungsabo lesen | One recurring invoice with line items and PDF attachments. |
| `create_invoice_subscription` | `write` | idempotent | Rechnungsabo anlegen | Creates a recurring invoice: on the due date a draft is created (default), or the invoice is issued, or issued and sent. |
| `update_invoice_subscription` | `write` | idempotent | Rechnungsabo ändern | Changes the given fields of a recurring invoice, for example quantities; applies from the next draft. |
| `create_draft_from_subscription` | `write` | idempotent | Entwurf aus Abo erzeugen | Creates the draft for the next date right away and moves the schedule on; there is exactly one draft per date. |
| `attach_invoice_draft_file` | `write` | idempotent | PDF an Entwurf hängen | Attaches a PDF to an open draft; it moves to the invoice when the draft is issued and goes out with every invoice e-mail. |
| `attach_invoice_subscription_file` | `write` | idempotent | PDF an Rechnungsabo hängen | Attaches a PDF to a recurring invoice: to every draft it creates, or only to the next one. |
| `attach_outgoing_invoice_file` | `write` | idempotent | PDF an Rechnung hängen | Attaches a PDF to an issued invoice; it goes out with every invoice e-mail, the invoice itself stays unchanged. |
| `list_outgoing_invoice_files` | `read` | read-only, idempotent | PDF-Anhänge einer Rechnung lesen | Name, size and SHA-256 of an invoice's attachments. |
| `delete_invoice_file` | `write` | destructive, idempotent | PDF-Anhang entfernen | Removes an attachment from a draft, an invoice or a recurring invoice; an attachment that was already sent is retained. |
| `list_mail_templates` | `read` | read-only, idempotent | Mail-Vorlagen lesen | Mail templates for invoice, quote and reminder e-mails. They are maintained in the app and can be chosen when sending. |

## Quotes and payment reminders

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_quotes` | `read` | read-only, idempotent | Angebote lesen | Quotes with totals and status. |
| `get_quote` | `read` | read-only, idempotent | Angebot lesen | One quote with its line items. |
| `create_quote` | `write` | idempotent | Angebot anlegen | Creates a quote with a consecutive quote number; nothing is posted. |
| `update_quote` | `write` | idempotent | Angebot ändern | Changes a quote that has not been converted into an invoice yet. |
| `set_quote_status` | `write` | idempotent | Angebotsstatus setzen | Sets the status of a quote: draft, sent, accepted, rejected or expired. |
| `create_draft_from_quote` | `write` | idempotent | Rechnungsentwurf aus Angebot | Creates an invoice draft from a quote. |
| `get_quote_document` | `read` | read-only, idempotent | Angebot als PDF | The quote as PDF (base64). |
| `send_quote` | `send` | destructive, idempotent, open-world | Angebot per E-Mail senden | E-mails the quote to the address stored for the business partner. |
| `list_dunning_candidates` | `read` | read-only, idempotent | Mahnvorschläge lesen | Reminder proposals: open invoices with due date, days overdue, previous reminder level and reminder block. Creates nothing. |
| `list_dunning_notices` | `read` | read-only, idempotent | Mahnungen lesen | Payment reminders created so far, newest first. |
| `create_dunning_notice` | `write` | none | Mahnung erzeugen | Creates the next reminder level for an open invoice; with `preview` only the proposal is shown. Nothing is sent. |
| `get_dunning_document` | `read` | read-only, idempotent | Mahnung als PDF | The payment reminder as PDF (base64). |
| `send_dunning_notice` | `send` | destructive, idempotent, open-world | Mahnung per E-Mail senden | E-mails the payment reminder to the address stored for the customer; a reminder block or a cancellation prevents sending. |

## Purchase invoices and receipts

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_incoming_invoices` | `read` | read-only, idempotent | Eingangsrechnungen | Purchase invoices, filterable by fiscal year and status. |
| `book_incoming_invoice` | `write` | idempotent | Eingangsrechnung buchen | Posts a purchase invoice with input VAT as a payable to the supplier; supports a dry run. |
| `book_incoming_invoices_batch` | `write` | idempotent | Eingangsrechnungen im Stapel buchen | Posts up to 50 purchase invoices one after another; each row has its own result. |
| `cancel_incoming_invoice` | `write` | idempotent | Eingangsrechnung stornieren | Takes back the posting of an unpaid purchase invoice by counter-entry; a linked receipt is back in review afterwards. |
| `decide_incoming_invoice_approval` | `write` | idempotent | Eingangsrechnung freigeben oder ablehnen | Approves or rejects a purchase invoice, or resets the decision; only an approved invoice can be paid by hand. |
| `list_receipts` | `read` | read-only, idempotent | Belege | Receipts with their booking status, newest first, however they arrived (API, upload, mail inbox or integration). |
| `get_receipt` | `read` | read-only, idempotent | Einzelner Beleg | One receipt with its booking status, including deleted ones. |
| `get_receipt_proposal` | `read` | read-only, idempotent | Buchungsvorschlag eines Belegs | The booking proposal for a receipt: accounts, tax key, amount, date, counterparty, confidence level, origin of the account assignment and whether something needs checking before booking. |
| `get_receipt_file` | `read` | read-only, idempotent | Originaldatei eines Belegs | Name, type and download path of a receipt's original file; the file itself is fetched through the REST API with the key. |
| `get_receipt_archive` | `read` | read-only, idempotent | Belegarchiv eines Geschäftsjahres | Receipt archive of a fiscal year (GoBD data export with the original receipts): its parts with receipt count, size and download path. |
| `upload_receipt` | `write` | idempotent | Beleg einliefern | Puts a receipt (PDF, PNG, JPEG, WebP, or XRechnung/ZUGFeRD XML) into the receipt inbox, where it runs through recognition and is booked by the autopilot or waits for review. |
| `create_receipt_upload_link` | `write` | idempotent | Hochlade-Link für Belege | Creates an upload link for original receipts of this company (valid 30 minutes, up to 20 files, no sign-in). Uploaded files land in the receipt inbox and run through recognition, duplicate check and autopilot. |
| `get_receipt_inbox_address` | `write` | idempotent | E-Mail-Adresse des Belegeingangs | The e-mail address of the company's receipt inbox, to which invoices can be forwarded. |
| `confirm_receipt` | `write` | idempotent | Beleg buchen | Books a receipt as in the app's receipt review, with all checks (fiscal year, period lock, duplicates, VAT, supplier, asset, accrual), optionally with a reasoned correction of the proposal. |
| `dismiss_receipt` | `write` | idempotent | Beleg nicht buchen | Files a receipt's open proposal as not to be booked, with a reason (for example duplicate or private expense); the receipt stays archived. |
| `discard_receipt` | `write` | idempotent | Beleg verwerfen | Discards an unbooked receipt, for example one delivered by mistake; the file is retained and the receipt can be restored in the app. |
| `delete_receipt` | `write` | destructive, idempotent | Beleg endgültig löschen | Permanently deletes a misdirected document with its file, only if it was never reviewed, discarded, booked or linked. |

## Bank, payments and open items

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_bank_accounts` | `read` | read-only, idempotent | Bankkonten lesen | Bank accounts with masked IBAN, ledger account and last retrieval; with a fiscal year also the statement balance and the book balance. |
| `list_bank_transactions` | `read` | read-only, idempotent | Bankumsätze | Bank transactions, filterable by date range and status. |
| `get_bank_transaction` | `read` | read-only, idempotent | Bankumsatz lesen | One bank transaction with bank details and ledger account. |
| `import_bank_statement` | `write` | idempotent | Kontoauszug importieren | Imports a bank statement file (CSV, CAMT.053 or MT940) for a bank account; account, balances and duplicates are checked. |
| `book_bank_transaction` | `write` | idempotent | Bankumsatz buchen | Books an open bank transaction through the app's booking path; for a company that keeps a balance sheet, expenses and income need the receipt, a cash-basis business books at the cash flow. An already booked transaction is corrected by reversal and rebooking in one step. |
| `match_bank_transaction` | `write` | idempotent | Bankzahlung zuordnen | Assigns the bank amount to an open purchase or sales invoice; partial payments leave the rest open; cash discount and payment differences are checked on the server. |
| `ignore_bank_transaction` | `write` | idempotent | Bankumsatz ignorieren | Hides a private or irrelevant unbooked transaction with a reason; it can be reopened in the app. |
| `list_open_items` | `read` | read-only, idempotent | Offene Posten | Open items (receivables and payables) with due date and remaining amount, filterable by kind and status. |
| `list_payments` | `read` | read-only, idempotent | Zahlungen und Ausgleiche | Clearings of open items (payments, cash discounts, bank matches) with their clearing ID. |
| `record_outgoing_invoice_payment` | `write` | idempotent | Zahlung zu Ausgangsrechnung erfassen | Records a payment for a sales invoice, including cash discount with VAT correction; accounts with bank retrieval are handled through bank matching instead. |
| `record_incoming_invoice_payment` | `write` | idempotent | Zahlung zu Eingangsrechnung erfassen | Records a payment for an approved purchase invoice, including cash discount with input-VAT correction. |
| `record_invoice_payments_batch` | `write` | idempotent | Zahlungen zu Rechnungen im Stapel erfassen | Records up to 50 payments for invoices one after another; each row has its own result. |
| `storno_payment` | `write` | idempotent | Zahlung stornieren | Takes back a recorded payment or clearing by counter-entry; the invoice is open again. |

## Cash book, fixed assets, provisions and accruals

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_cash_book` | `read` | read-only, idempotent | Kassenbuch lesen | Reads the cash book. |
| `book_cash_entry` | `write` | idempotent | Kasseneinzahlung oder Kassenauszahlung buchen | Posts a cash deposit or a cash withdrawal. |
| `reverse_cash_entry` | `write` | destructive, idempotent | Kassenbuchung stornieren | Reverses a cash-book entry by counter-entry. |
| `list_fixed_assets` | `read` | read-only, idempotent | Anlagenverzeichnis | Fixed-asset register with acquisition, depreciation method and useful life. |
| `create_fixed_asset` | `write` | idempotent | Anlagegut ins Verzeichnis aufnehmen | Adds a fixed asset to the register, including existing assets with accumulated depreciation; creates no acquisition posting. |
| `dispose_fixed_asset` | `write` | idempotent | Anlagenabgang buchen | Posts the disposal of a fixed asset: disposal date, write-off of the residual book value and the depreciation due. |
| `book_fixed_asset_movement` | `write` | idempotent | Anlagenbewegung oder Nebenkosten buchen | Posts an asset movement or incidental acquisition costs. |
| `run_depreciation` | `write` | idempotent | AfA-Lauf buchen | Posts the fiscal year's depreciation as in the app, optionally spread over months or quarters; periods already posted are taken into account. |
| `list_provisions` | `read` | read-only, idempotent | Rückstellungen lesen | Reads the provisions. |
| `create_provision` | `write` | idempotent | Rückstellung bilden | Creates a provision. |
| `release_provision` | `write` | idempotent | Rückstellung auflösen | Releases a provision. |
| `use_provision` | `write` | idempotent | Rückstellung in Anspruch nehmen | Uses a provision. |
| `list_accruals` | `read` | read-only, idempotent | Rechnungsabgrenzungen lesen | Reads the accruals and deferrals (Rechnungsabgrenzung). |
| `create_accrual` | `write` | idempotent | Rechnungsabgrenzung bilden | Creates an accrual or deferral. |
| `release_accrual` | `write` | idempotent | Rechnungsabgrenzung auflösen | Releases an accrual or deferral. |

## VAT preview, EÜR and period lock

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `get_vat_return` | `read` | read-only, idempotent | UStVA-Vorschau | Preview of the advance VAT return for a month or quarter: box figures, permanent extension and special prepayment. Nothing is transmitted. |
| `get_vat_supplementary` | `read` | read-only, idempotent | UStVA: Fragen des Vordrucks | The conditional questions of the official advance VAT return form for a period, and whether they are confirmed. |
| `confirm_vat_supplementary` | `write` | idempotent | UStVA: keine Sonderfälle bestätigen | Records the user's answer that none of the form's conditional cases applies in a period. |
| `get_euer` | `read` | read-only, idempotent | Anlage-EÜR-Vorschau | Preview of the Anlage EÜR of a fiscal year (income, expenses, profit, notes) for cash-basis businesses. Stores nothing. |
| `list_bookkeeping_periods` | `read` | read-only, idempotent | Festschreibungsstatus | Lock status of every month, optionally for one fiscal year. |
| `finalize_bookkeeping_period` | `finalize` | destructive, idempotent | Zeitraum festschreiben | Locks the books irrevocably up to a date (Festschreibung under GoBD), after the user's explicit confirmation. |

## Tax filings

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `list_tax_deadlines` | `read` | read-only, idempotent | Steuerfristen eines Jahres | The tax deadlines of a year as in the app's tax calendar, each with due date, done and overdue. |
| `list_tax_filings` | `read` | read-only, idempotent | Steuermeldungen und Übermittlungsstand | The company's tax filings and returns (advance VAT return, EC sales list, permanent extension, special prepayment, annual VAT return, E-Bilanz, corporate tax, trade tax, Anlage EÜR) with status and transfer ticket. |
| `check_tax_filing` | `write` | idempotent | Amtlich prüfen (ohne Versand) | Runs the official ELSTER check (ERiC, test flag) on a filing. Nothing is transmitted. |
| `create_tax_draft` | `write` | idempotent | Entwurf aus den Büchern anlegen (ZM, Dauerfristverlängerung, E-Bilanz) | Creates a draft from the books: the EC sales list of a period, the permanent extension for the current year, or the empty E-Bilanz of a fiscal year. Nothing is transmitted and an existing draft is never overwritten. |
| `prepare_tax_filing` | `write` | none | Übermittlung vorbereiten | Prepares a transmission: freezes the values and their SHA-256 checksum for 30 minutes, with the official check and any blockers. Nothing is sent. |
| `get_tax_operation` | `read` | read-only, idempotent | Stand eines Steuer-Vorgangs | A prepared operation with the current transmission status of the filing. |
| `request_tax_filing_approval` | `write` | idempotent | Freigabe-Link für die App | The standard way to file: returns a link (valid 7 days) under which the user opens the prepared operation in the app, compares the values and transmits with their own PIN. |
| `submit_tax_filing` | `file` | destructive, open-world | An das Finanzamt übermitteln (nicht rückholbar) | Transmits a prepared operation to the tax office with the user's PIN. Needs the right `file`, which is not granted at present, so the approval link is the way to file. |

## Annual financial statements and disclosure

| Tool | Right | Annotations | Title (German) | What it does |
| --- | --- | --- | --- | --- |
| `publish_trial_balance` | `write` | idempotent | Saldenliste übergeben | Hands the trial balance from the books over to the annual financial statements and returns mapping counts and notes. |
| `upload_trial_balance` | `write` | idempotent | Saldenliste mit eigenen Zahlen übergeben | Hands over figures for the annual financial statements from another bookkeeping system, a spreadsheet or a trial balance: account number, name and closing balance per account; the sum must be zero. |
| `get_annual_report_status` | `read` | read-only, idempotent | Stand des Jahresabschlusses | Where a fiscal year stands: whether the annual financial statements are unlocked, the next step and the page in the app, lock date, trial balance handed over, balance sheet and profit-and-loss confirmed, issued statements, disclosure files, E-Bilanz and corporate and trade tax of the year. |
| `list_annual_reports` | `read` | read-only, idempotent | Ausgestellte Jahresabschlüsse | Issued annual financial statements (as prepared and for disclosure, PDF or DOCX) with ID and checksum. |
| `get_annual_report` | `read` | read-only, idempotent | Inhalt eines Jahresabschlusses | Balance sheet, profit-and-loss account, notes, management report and further sections of an issued annual report as text, exactly the issued version. |
| `download_annual_report` | `read` | read-only, idempotent | Datei eines Jahresabschlusses | Names how to download the PDF or DOCX file of an issued annual report; the file does not pass through the model. |
| `list_disclosure_files` | `read` | read-only, idempotent | Offenlegungsdateien | Archived disclosure files for the company register with document type, target, file name and SHA-256. |
| `download_disclosure_file` | `read` | read-only, idempotent | Offenlegungsdatei (XML) | The exact bytes of a disclosure file as XML text; the SHA-256 from the list verifies them. |
