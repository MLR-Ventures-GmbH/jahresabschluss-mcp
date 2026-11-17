# Jahresabschluss.io MCP server

A hosted [Model Context Protocol](https://modelcontextprotocol.io/) server for the books of a German business at [jahresabschluss.io](https://jahresabschluss.io/en): bookkeeping, invoices, bank, receipts, fixed assets, cash-basis accounting (EÜR), tax filings, annual financial statements and disclosure files.

This repository explains how to connect an AI assistant such as Claude, ChatGPT, Codex or Cursor. It holds documentation and the registry entry only. The server itself is hosted, so there is nothing to install and no platform source code here.

[Deutsche Fassung weiter unten](#deutsch)

| | |
| --- | --- |
| Endpoint | `https://jahresabschluss.io/mcp` |
| Transport | Streamable HTTP |
| Authentication | OAuth 2.0 (authorization code with PKCE, dynamic client registration), or a per-company API key as bearer token |
| Registry name | `io.jahresabschluss/buchhaltung` in the [official MCP Registry](https://registry.modelcontextprotocol.io/); a copy of the entry is in [`server.json`](server.json) |
| Tools | 125 as of 17 November 2026, listed in [docs/tools.md](docs/tools.md) |
| Operator | MLR Ventures GmbH, Germany |

## What an assistant can do

A connected assistant works in the real books of one company, the same ledger the web app uses. Every write runs through the same booking core as the app.

- **Ledger:** chart of accounts (SKR03 or SKR04), journal, trial balance (Summen- und Saldenliste), open items, opening balances, exports.
- **Sales:** invoices as PDF, ZUGFeRD or XRechnung, drafts, recurring invoices, quotes, credit notes and payment reminders.
- **Purchases and receipts:** purchase invoices and their approval, receipts with booking proposals, an upload link and a receipt e-mail address for the original documents.
- **Bank and payments:** bank accounts and transactions, statement import (CSV, CAMT.053, MT940), matching payments to open invoices.
- **Sub-ledgers:** cash book, fixed-asset register with depreciation run, provisions, accruals and deferrals.
- **VAT and EÜR:** preview of the advance VAT return, and of the Anlage EÜR for cash-basis businesses.
- **Tax filings:** deadlines and filing status, the official ELSTER check without transmission, and the preparation of the advance VAT return, EC sales list, permanent extension (Dauerfristverlängerung), annual VAT return, E-Bilanz, corporate tax, trade tax and Anlage EÜR. The assistant checks and prepares; the user transmits in the app through an approval link.
- **Annual financial statements:** where a fiscal year stands, the issued statements (balance sheet, profit-and-loss account, notes, management report) as text, their files, and the disclosure files for the company register.

What stays in the web app:

- Transmitting to the tax office, through the approval link and with the user's own ELSTER PIN.
- Preparing, reviewing and issuing the annual financial statements, and approving the disclosure: after that approval the software files the statements directly with the company register (Unternehmensregister); the XML file can still be downloaded and filed by hand.
- Bank connections, company settings and the bank details (IBAN, BIC) of business partners.
- Creating API keys, which needs the second factor.

## Who it is for

Businesses in Germany that keep their books at jahresabschluss.io: corporations (GmbH, UG, AG, SE, KGaA), cooperatives (eG), partnerships (OHG, KG, GmbH & Co. KG, GbR, PartG), sole traders and freelancers, with double-entry bookkeeping on SKR03 or SKR04 or with cash-basis accounting (EÜR).

You need an account with a company at jahresabschluss.io, the Administrator or Preparer role in its workspace, and your second factor. Which tools a connection sees depends on the modules the company uses:

- Bookkeeping or EÜR module: all tools.
- Tax module only: the tax tools and the company's master data.
- Annual financial statements only: handing over the trial balance and reading the status of the statements.

## Connect

One connection always covers exactly one company. For a second company, connect again.

### With sign-in (OAuth): Claude, ChatGPT and other clients with custom connectors

No key to copy.

1. In your AI program, add a custom connector (custom MCP server) with the address `https://jahresabschluss.io/mcp` and OAuth as authentication.
2. The program opens jahresabschluss.io. Sign in, choose the company, the rights (Read is always included; Post and Send invoices by e-mail are your choice) and the validity (30 days, 90 days or 1 year). Click Allow and confirm with your second factor.
3. Try it: "Show me the last five journal entries."

Where the setting lives, according to each vendor's own help pages (menu names change; if your screen differs, follow the vendor's current help):

- **Claude** (claude.ai in the browser, the desktop app, and the mobile apps once connected): Customize › Connectors › + Add › Add custom connector. If Claude asks how it should identify itself, choose "Register automatically". On Team and Enterprise plans an owner adds the connector under Organization settings › Connectors and each member connects individually. [Guide](https://jahresabschluss.io/en/ki-agent/claude-ai-connector-german-bookkeeping)
- **ChatGPT** (in the browser): chatgpt.com/plugins › "+" › "Add custom MCP server", authentication OAuth, then "Create as a plugin". Whether your plan offers custom connections, and whether ChatGPT may also write, is decided by OpenAI. [Guide](https://jahresabschluss.io/en/ki-agent/chatgpt-connector-german-bookkeeping)
- **Other clients** that support OAuth for remote MCP servers connect the same way. The server registers clients automatically; you never create a client ID.

As of 8 November 2026 the sign-in route has been tested by hand in Claude; for the other clients it follows the vendors' documentation.

### With an API key: Codex, Claude Code, Cursor and other developer tools

Create the key in the app: Bookkeeping › Settings › "AI agent & API" (German interface: „KI-Agent & API“) › New key. A key belongs to exactly one company, carries the rights you tick (read is always included; write, send, finalize and webhooks are optional), expires after 30 days, 90 days or one year, and can be revoked at any time. It is shown once and stored only as a hash. Keep it in an environment variable or in the client's own configuration, never in a chat and never in a repository.

**Codex** (according to OpenAI, the CLI, the IDE extension and the ChatGPT desktop app share `~/.codex/config.toml`):

```bash
# in your shell profile, never in a repository
export JAB_API_KEY="jab_live_…"

codex mcp add buchhaltung \
  --url https://jahresabschluss.io/mcp \
  --bearer-token-env-var JAB_API_KEY
```

```toml
# ~/.codex/config.toml
[mcp_servers.buchhaltung]
url = "https://jahresabschluss.io/mcp"
bearer_token_env_var = "JAB_API_KEY"
# ask before every tool that is not marked read-only
default_tools_approval_mode = "writes"
```

**Claude Code:**

```bash
claude mcp add --transport http --scope user buchhaltung \
  https://jahresabschluss.io/mcp \
  --header "Authorization: Bearer jab_live_…"
```

`--scope user` keeps the entry in your personal configuration. Do not use `--scope project`: it writes `.mcp.json` into the repository.

**Cursor** (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "buchhaltung": {
      "url": "https://jahresabschluss.io/mcp",
      "headers": { "Authorization": "Bearer ${env:JAB_API_KEY}" }
    }
  }
}
```

**Claude desktop app on Mac and Windows:** besides the connector above there is the extension "Buchhaltung für Claude", installed with a double-click and one key. [Guide and download](https://jahresabschluss.io/en/ki-agent/claude-desktop-extension-bookkeeping)

**Other clients:** any client that reaches a Streamable HTTP server and sends an `Authorization: Bearer` header works the same way; the key is also accepted in an `X-API-Key` header. A client that only starts local stdio servers can use the published `mcp-remote` bridge, with the key in the `env` block and not in the arguments:

```json
{
  "mcpServers": {
    "buchhaltung": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://jahresabschluss.io/mcp",
        "--header", "Authorization:${JAB_AUTH}"],
      "env": { "JAB_AUTH": "Bearer jab_live_…" }
    }
  }
}
```

## Tool groups

| Group | Examples | Rights |
| --- | --- | --- |
| Company, accounts and ledger | `get_saldenliste`, `list_journal_entries`, `create_journal_entry`, `storno_journal_entry`, `export_bookkeeping` | read, write |
| Business partners | `list_business_partners`, `create_business_partner` | read, write |
| Sales invoices, drafts and recurring invoices | `create_invoice_draft`, `create_outgoing_invoice`, `send_outgoing_invoice`, `cancel_outgoing_invoice` | read, write, send |
| Quotes and payment reminders | `create_quote`, `list_dunning_candidates`, `send_dunning_notice` | read, write, send |
| Purchase invoices and receipts | `create_receipt_upload_link`, `get_receipt_proposal`, `confirm_receipt`, `book_incoming_invoice` | read, write |
| Bank, payments and open items | `list_bank_transactions`, `import_bank_statement`, `match_bank_transaction`, `list_open_items` | read, write |
| Cash book, fixed assets, provisions and accruals | `book_cash_entry`, `create_fixed_asset`, `run_depreciation`, `create_provision` | read, write |
| VAT preview, EÜR and period lock | `get_vat_return`, `get_euer`, `finalize_bookkeeping_period` | read, write, finalize |
| Tax filings | `list_tax_deadlines`, `check_tax_filing`, `prepare_tax_filing`, `request_tax_filing_approval` | read, write |
| Annual financial statements and disclosure | `publish_trial_balance`, `get_annual_report_status`, `get_annual_report`, `download_disclosure_file` | read, write |

The full list with every tool, its right and its annotations is in [docs/tools.md](docs/tools.md).

Example prompts:

- "Book the new receipts and ask me about the ones that are not clear."
- "Match October's payments to the open invoices. Who hasn't paid yet?"
- "Prepare the VAT return for September and give me the approval link."

Tool titles, descriptions and several response fields are German. The books follow German rules (SKR03 or SKR04, German VAT, GoBD); the conversation can be in English.

## Rights and safety

- **One company per connection.** The company follows from the approval or the key, never from a tool argument.
- **Rights.** `read` is part of every connection. `write` posts, issues and prepares; `send` e-mails invoices, quotes and payment reminders. Sign-in offers these three. An API key can also carry `finalize` (locking periods) and `webhooks` (managed through the REST API). A connection only sees the tools its rights allow, so a read-only connection cannot change anything.
- **Expiry and revocation.** A sign-in approval lasts 30 days, 90 days or 1 year and is not renewed silently; a key expires after one year at most. Every connection appears under "AI agent & API" in the app and can be revoked there at any time.
- **Confirmation before anything that cannot be undone.** The server instructs the assistant to show the details and get the user's explicit yes before it issues or e-mails an invoice, sends a quote or a payment reminder, reverses or cancels something, issues a credit note, approves or rejects a purchase invoice, deletes a receipt or locks a period. One yes covers one action. Every tool carries the MCP annotations for read-only, destructive, idempotent and open-world behaviour, so clients can ask before any call that is not read-only. Instructions guide the model; the hard limits are the rights of the connection and the checks on the server.
- **Same rules as the app.** Debits equal credits, corrections are reversals and never deletions, nothing is posted into a locked period, receipts are checked for duplicates. Repeating a request with the same idempotency key does not post twice.
- **Tax office.** The assistant runs the official ELSTER check and prepares the filing. The user transmits in the app through the approval link, with their own ELSTER PIN, which never goes into the chat. A right that would let an assistant transmit by itself is not granted at present.
- **Original receipts only.** A file attached in a chat does not reach the server. The assistant hands out an upload link (valid 30 minutes, up to 20 files) or the company's receipt e-mail address. The original then lands in the receipt inbox: clear receipts are booked by the autopilot, as with receipts sent by e-mail; for the rest the assistant shows the proposal and books after the user's yes.
- **Third-party text is data.** Bank references, supplier names and receipt contents come from third parties. The server marks them as data and tells the assistant never to follow instructions found in them.
- **Keys.** Never paste a key into a chat. Start with a read-only key with a short expiry and add rights when you trust the routine.
- **Sandbox.** A key created in a company's sandbox works on a copy of the books; sending and transmitting are blocked there.
- **No free queries.** The tool list is fixed; there is no SQL and no table access.
- **Your AI provider.** Data reaches an AI program only when you connect it. What the assistant reads is processed by the provider of that program under your terms with them. See the [privacy policy](https://jahresabschluss.io/datenschutz).

The software does not replace tax or legal advice or an audit. What an assistant produces is a proposal; responsibility for the books stays with the company.

## Technical notes

- The MCP server is a thin adapter over the REST API of the bookkeeping: same key, same rights, same checks. See the API documentation in [German](https://jahresabschluss.io/buchhaltung-api) or [English](https://jahresabschluss.io/en/ki-agent/bookkeeping-rest-api).
- JSON-RPC over HTTP POST with JSON responses. The server does not open a server-initiated stream, so `GET` answers 405.
- Protocol revisions 2025-06-18, 2025-03-26 and 2024-11-05. The server offers tools and sends instructions with `initialize`; it has no resources or prompts.
- A request without credentials answers 401 with `WWW-Authenticate: Bearer resource_metadata="https://jahresabschluss.io/.well-known/oauth-protected-resource/mcp"`. The authorization server metadata is at `https://jahresabschluss.io/.well-known/oauth-authorization-server`.
- OAuth: authorization code with PKCE (S256), dynamic client registration, public and confidential clients, scopes `read`, `write` and `send`. There is no refresh token; when access expires, the user connects again.
- Limit: 120 requests per minute per key.
- Files such as the DATEV-format posting batch, the GoBD package or the PDF of an annual report do not pass through the model; the tools name the download path of the REST API instead.

## Documentation

- [Connect an AI agent to German accounting](https://jahresabschluss.io/en/ki-agent/connect-ai-agent-to-german-accounting) (English overview)
- [MCP server for your accounting](https://jahresabschluss.io/en/ki-agent/mcp-server-accounting)
- [Claude connector](https://jahresabschluss.io/en/ki-agent/claude-ai-connector-german-bookkeeping) · [ChatGPT connector](https://jahresabschluss.io/en/ki-agent/chatgpt-connector-german-bookkeeping) · [Codex](https://jahresabschluss.io/en/ki-agent/bookkeeping-with-codex) · [Cursor (German)](https://jahresabschluss.io/ki-agent/cursor-buchhaltung)
- [Tax filings with your own AI agent](https://jahresabschluss.io/en/ki-agent/ai-agent-german-taxes) · [Annual financial statements via API and MCP](https://jahresabschluss.io/en/ki-agent/german-annual-report-api)
- [Sandbox for accounting agents](https://jahresabschluss.io/en/ki-agent/sandbox-for-accounting-agents) · [Prompt injection through invoices and bank references](https://jahresabschluss.io/en/ki-agent/prompt-injection-accounting-agents)
- German: [KI-Agent anbinden](https://jahresabschluss.io/ki-agent) · [Claude und ChatGPT anbinden](https://jahresabschluss.io/claude-chatgpt-anbinden) · [Buchhaltung-API](https://jahresabschluss.io/buchhaltung-api)

## Operator

Jahresabschluss.io is operated by MLR Ventures GmbH, Germany (commercial register: Amtsgericht Charlottenburg, HRB 284829 B).

[Legal notice](https://jahresabschluss.io/impressum) · [Privacy policy](https://jahresabschluss.io/datenschutz) · [Security](https://jahresabschluss.io/security) · [Contact](https://jahresabschluss.io/contact)

Product names of AI programs are trademarks of their owners. Naming a program here describes how to connect it and implies no endorsement.

---

## Deutsch

Ein gehosteter MCP-Server (Model Context Protocol) für die Bücher eines Unternehmens bei [jahresabschluss.io](https://jahresabschluss.io/): Buchhaltung, Rechnungen, Bank, Belege, Anlagen, EÜR, Steuermeldungen, Jahresabschlüsse und Offenlegungsdateien.

Dieses Repository beschreibt, wie Sie ein KI-Programm wie Claude, ChatGPT, Codex oder Cursor anbinden. Es enthält nur die Dokumentation und den Eintrag in der MCP Registry. Der Server wird von uns betrieben; zu installieren ist nichts, und Quellcode der Plattform liegt hier nicht.

| | |
| --- | --- |
| Adresse | `https://jahresabschluss.io/mcp` |
| Übertragung | Streamable HTTP |
| Anmeldung | OAuth 2.0 (Autorisierungscode mit PKCE, automatische Registrierung des Programms) oder ein Schlüssel je Gesellschaft als Bearer-Token |
| Name in der Registry | `io.jahresabschluss/buchhaltung` in der [offiziellen MCP Registry](https://registry.modelcontextprotocol.io/), Abschrift in [`server.json`](server.json) |
| Werkzeuge | 125 zum Stand 17.11.2026, aufgelistet in [docs/tools.md](docs/tools.md) |
| Betreiber | MLR Ventures GmbH |

### Was ein KI-Programm damit kann

Das angebundene Programm arbeitet in den echten Büchern einer Gesellschaft. Gebucht wird über denselben Buchungskern wie in der App.

- **Hauptbuch:** Kontenrahmen (SKR03 oder SKR04), Journal, Summen- und Saldenliste, offene Posten, Eröffnungswerte, Exporte.
- **Ausgangsseite:** Rechnungen als PDF, ZUGFeRD oder XRechnung, Entwürfe, wiederkehrende Rechnungen, Angebote, Gutschriften und Mahnungen.
- **Eingangsseite und Belege:** Eingangsrechnungen und ihre Freigabe, Belege mit Buchungsvorschlag, Hochlade-Link und Beleg-Adresse für die Originale.
- **Bank und Zahlungen:** Bankkonten und Bankumsätze, Import von Kontoauszügen (CSV, CAMT.053, MT940), Zuordnung von Zahlungen zu offenen Rechnungen.
- **Nebenbücher:** Kassenbuch, Anlagenverzeichnis mit AfA-Lauf, Rückstellungen und Rechnungsabgrenzungen.
- **Umsatzsteuer und EÜR:** Vorschau der Umsatzsteuer-Voranmeldung und der Anlage EÜR.
- **Steuermeldungen:** Fristen und Übermittlungsstand, die amtliche ELSTER-Prüfung ohne Versand und die Vorbereitung von Umsatzsteuer-Voranmeldung, Zusammenfassender Meldung, Dauerfristverlängerung, Umsatzsteuer-Jahreserklärung, E-Bilanz, Körperschaftsteuer, Gewerbesteuer und Anlage EÜR. Das Programm prüft und bereitet vor; übermittelt wird von Ihnen in der App über einen Freigabe-Link.
- **Jahresabschluss:** Stand eines Geschäftsjahres, ausgestellte Abschlüsse (Bilanz, GuV, Anhang, Lagebericht) als Text, ihre Dateien und die Offenlegungsdateien für das Unternehmensregister.

In der App bleiben: die Übermittlung an das Finanzamt (über den Freigabe-Link, mit Ihrer ELSTER-PIN), das Erstellen, Prüfen und Ausstellen des Jahresabschlusses und die Freigabe der Offenlegung (danach übermittelt die Software die Fassung direkt an das Unternehmensregister; die XML-Datei können Sie weiterhin selbst einreichen), Bankanbindungen, Einstellungen der Gesellschaft und Bankverbindungen von Geschäftspartnern sowie das Anlegen von Schlüsseln mit dem zweiten Faktor.

### Für wen

Für Unternehmen in Deutschland, die ihre Bücher bei jahresabschluss.io führen: Kapitalgesellschaften (GmbH, UG, AG, SE, KGaA), Genossenschaften (eG), Personengesellschaften (OHG, KG, GmbH & Co. KG, GbR, PartG), Einzelunternehmen und Freiberufler, mit doppelter Buchführung nach SKR03 oder SKR04 oder mit Einnahmen-Überschuss-Rechnung.

Sie brauchen ein Konto mit einer Gesellschaft bei jahresabschluss.io, die Rolle Administrator oder Ersteller im Arbeitsbereich und Ihren zweiten Faktor. Welche Werkzeuge eine Verbindung sieht, hängt von den Modulen der Gesellschaft ab: mit Buchhaltung oder EÜR alle, nur mit dem Steuer-Modul die Steuer-Werkzeuge und die Stammdaten, nur mit dem Jahresabschluss die Übergabe der Saldenliste und der Stand des Abschlusses.

### Verbinden

Eine Verbindung gilt immer für genau eine Gesellschaft. Für eine weitere Gesellschaft verbinden Sie erneut.

**Mit Anmeldung (OAuth): Claude, ChatGPT und andere Programme mit eigenen Konnektoren.** Ohne Schlüssel zum Kopieren.

1. Im KI-Programm einen eigenen Konnektor (eigenen MCP-Server) mit der Adresse `https://jahresabschluss.io/mcp` anlegen, Anmeldung OAuth.
2. Das Programm öffnet jahresabschluss.io. Sie melden sich an und wählen die Gesellschaft, die Rechte (Lesen ist immer dabei; Buchen und E-Mail-Versand nach Wahl) und die Gültigkeit (30 Tage, 90 Tage oder 1 Jahr). Dann erlauben Sie und bestätigen mit Ihrem zweiten Faktor.
3. Zum Test: „Zeig mir die letzten fünf Buchungen.“

Wo die Einstellung steht, beschreiben die Anbieter in ihrer eigenen Hilfe; die Menüs ändern sich. In Claude (claude.ai im Browser, die App am Rechner und danach auch auf dem Telefon): „Anpassen“ › „Konnektoren“ › eigenen Konnektor hinzufügen. In ChatGPT im Browser: chatgpt.com/plugins › „+“ › „Add custom MCP server“, Anmeldung OAuth. Ob Ihr ChatGPT-Tarif eigene Verbindungen zulässt und ob ChatGPT auch buchen darf, legt OpenAI fest. Von Hand geprüft ist die Anmeldung zum Stand 08.11.2026 in Claude; für die übrigen Programme folgt sie der Dokumentation der Anbieter.

**Mit Schlüssel: Codex, Claude Code, Cursor und andere Entwicklerprogramme.** Den Schlüssel erstellen Sie in der App unter Buchhaltung › Einstellungen › „KI-Agent & API“ › „Neuer Schlüssel“. Er gilt für genau eine Gesellschaft, trägt die Rechte, die Sie wählen (Lesen ist immer dabei; Buchen, E-Mail-Versand, Festschreiben und Webhooks nach Wahl), läuft nach 30 Tagen, 90 Tagen oder einem Jahr ab und lässt sich jederzeit widerrufen. Er wird einmal angezeigt und nur als Prüfwert gespeichert. Er gehört in eine Umgebungsvariable oder in die Konfiguration des Programms, nie in den Chat und nie in ein Repository.

Die Konfigurationen für Codex, Claude Code, Cursor und die Brücke `mcp-remote` stehen oben im englischen Teil; die Adresse ist immer `https://jahresabschluss.io/mcp`, der Schlüssel steht im Kopf `Authorization: Bearer jab_live_…`. Für die Claude-App auf Mac und Windows gibt es zusätzlich die Erweiterung „Buchhaltung für Claude“ zum Doppelklicken ([Anleitung](https://jahresabschluss.io/claude-chatgpt-anbinden)).

### Beispiele

- „Buche die neuen Belege im Belegeingang.“
- „Gleiche die Bankumsätze dieses Monats ab. Wer hat noch nicht bezahlt?“
- „Prüfe die Umsatzsteuer-Voranmeldung und schick mir den Freigabe-Link.“

Die Werkzeuggruppen stehen in der Tabelle oben, die vollständige Liste in [docs/tools.md](docs/tools.md).

### Rechte und Sicherheit

- **Eine Gesellschaft je Verbindung.** Sie ergibt sich aus der Freigabe oder dem Schlüssel, nie aus dem, was das KI-Programm schreibt.
- **Rechte.** Lesen ist immer dabei. Buchen und E-Mail-Versand wählen Sie; ein Schlüssel kann zusätzlich Festschreiben und Webhooks tragen. Eine Verbindung sieht nur die Werkzeuge, die ihre Rechte erlauben. Mit einer Verbindung nur zum Lesen ändert das Programm nichts.
- **Ablauf und Widerruf.** Eine Freigabe gilt 30 Tage, 90 Tage oder 1 Jahr und verlängert sich nicht von selbst; ein Schlüssel läuft nach höchstens einem Jahr ab. Jede Verbindung steht in der App unter „KI-Agent & API“ und lässt sich dort jederzeit widerrufen.
- **Bestätigung vor dem Unumkehrbaren.** Der Server weist das Programm an, Ihnen die Einzelheiten zu zeigen und Ihr ausdrückliches Ja einzuholen, bevor es eine Rechnung ausstellt oder versendet, ein Angebot oder eine Mahnung sendet, etwas storniert, eine Gutschrift ausstellt, eine Eingangsrechnung freigibt oder ablehnt, einen Beleg löscht oder einen Zeitraum festschreibt. Ein Ja gilt für einen Vorgang. Jedes Werkzeug trägt die MCP-Hinweise, ob es nur liest, nicht rückholbar ist oder nach außen wirkt. Anweisungen leiten das Modell; die harten Grenzen sind die Rechte der Verbindung und die Prüfungen des Servers.
- **Dieselben Regeln wie in der App.** Soll gleich Haben, Storno statt Löschen, keine Buchung in festgeschriebene Zeiträume, Dublettenprüfung bei Belegen. Eine wiederholte Anfrage mit demselben Idempotenzschlüssel bucht kein zweites Mal.
- **Finanzamt.** Das Programm prüft amtlich und bereitet vor. Übermittelt wird von Ihnen in der App über den Freigabe-Link, mit Ihrer ELSTER-PIN; die PIN gehört nie in den Chat.
- **Belege nur aus dem Original.** Eine Datei im Chat erreicht den Server nicht. Das Programm gibt Ihnen einen Hochlade-Link (30 Minuten, bis 20 Dateien) oder die Beleg-Adresse der Gesellschaft. Das Original liegt dann im Belegeingang: eindeutige Belege bucht der Autopilot wie beim Posteingang, für die übrigen zeigt das Programm den Vorschlag und bucht nach Ihrem Ja.
- **Fremde Texte sind Daten.** Verwendungszwecke, Lieferantennamen und Beleginhalte stammen von Dritten. Der Server kennzeichnet sie als Daten und weist das Programm an, darin stehenden Aufforderungen nie zu folgen.
- **Schlüssel.** Nie in den Chat kopieren. Beginnen Sie mit einem Schlüssel nur zum Lesen und kurzer Laufzeit.
- **Sandbox.** Ein Schlüssel aus der Sandbox einer Gesellschaft bucht nur in der Kopie; Versand und Übermittlung sind dort gesperrt.
- **Ihr KI-Anbieter.** Daten erreichen ein KI-Programm nur, wenn Sie es anbinden. Was es liest, verarbeitet dessen Anbieter nach Ihren Bedingungen mit ihm. Mehr in der [Datenschutzerklärung](https://jahresabschluss.io/datenschutz).

Die Software ersetzt keine Steuer- oder Rechtsberatung und keine Abschlussprüfung. Was ein KI-Programm liefert, ist ein Vorschlag; die Verantwortung für die Buchführung bleibt beim Unternehmen.

### Dokumentation

[KI-Agent anbinden](https://jahresabschluss.io/ki-agent) · [Claude und ChatGPT anbinden](https://jahresabschluss.io/claude-chatgpt-anbinden) · [Buchhaltung-API](https://jahresabschluss.io/buchhaltung-api) · [MCP-Server für die Buchhaltung](https://jahresabschluss.io/ki-agent/mcp-server-buchhaltung) · [Steuern per KI-Agent](https://jahresabschluss.io/ki-agent/steuern-mit-ki-agent) · [Jahresabschluss per KI-Agent](https://jahresabschluss.io/ki-agent/jahresabschluss-per-ki-agent)

### Betreiber

Jahresabschluss.io wird von der MLR Ventures GmbH betrieben (Amtsgericht Charlottenburg, HRB 284829 B).

[Impressum](https://jahresabschluss.io/impressum) · [Datenschutz](https://jahresabschluss.io/datenschutz) · [Sicherheit](https://jahresabschluss.io/security) · [Kontakt](https://jahresabschluss.io/contact)

Produktnamen von KI-Programmen sind Marken ihrer Inhaber. Sie werden hier genannt, um die Anbindung zu beschreiben; eine Empfehlung durch die Anbieter ist damit nicht verbunden.
