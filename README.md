# BW-basic-dedupe

A single HTML file that reads a Bitwarden JSON export, groups entries that are copies of each other, and helps you decide which copy to keep — giving priority to the one holding a passkey or an authenticator code. It produces a deletion list you work through inside Bitwarden or use the updated JSON to import to a freshly emptied vault. It never directly changes your vault.


## How it handles your data

A plain Bitwarden export is a file full of usable credentials in clear text. Anything that asks you to open one deserves scrutiny, so here is exactly what this does:

- **It is one file with no dependencies.** No build step, no package manager, no framework, no fonts or images fetched from a CDN.
- **It makes no network requests.** The page ships a content security policy of `default-src 'none'; connect-src 'none'`, so the browser refuses outbound requests from it regardless of what the code does.
- **Your file never leaves the machine.** It is read with `FileReader` and held in memory in the tab. Nothing is written to disk except the files you explicitly download, and nothing is stored in `localStorage`, cookies, or anywhere else that survives a refresh.
- **Passwords are never displayed.** A group shows the shared password's length and nothing more. Usernames are shown, since you need them to tell entries apart.

### Verifying that for yourself

Don't take the paragraph above on faith — it is cheap to check:

- Open the file in a text editor. It is about 850 lines including the CSS, and the only `fetch`, `XMLHttpRequest`, or external `src` in it is none of them.
- Load the page, then turn off your network adapter and use it end to end. It works unchanged.
- Watch the Network tab in your browser's developer tools while you load an export. It stays empty.

## Getting an export

In Bitwarden: **Tools → Export vault → File format: .json**. Choose the plain `.json` format, not the password-protected one — an encrypted export cannot be read here.

Delete the export, and any delete list you download, as soon as you are finished with them.

## Using it

1. Download `vault-duplicate-review.html` from [Releases](../../releases) and open it in your browser. No server needed; `file://` is fine.
2. Choose or drop your export. Duplicate groups appear, with a keeper preselected in each.
3. Adjust the picks. Each group has a **Keep this** control per row and a **Delete** checkbox, plus an **Allow multiple credentials to be kept** toggle if one keeper isn't enough.
4. Download or copy the delete list, then remove those entries in Bitwarden.

## How entries are matched

Two entries are copies when the **site**, the **username** and the **password** are all identical.

- **Site** defaults to the registrable domain, so `amazon.com`, `www.amazon.com` and `smile.amazon.com` group together. A toggle switches to exact hostname matching.
- **Usernames** are compared case-insensitively, after trimming whitespace.
- An entry with **several saved URIs** joins a group if any one of its domains matches.
- An entry with **no URI at all** falls back to matching on its name, and is labelled as such in the group header.
- Entries with **no stored password** are left out of matching and counted in a notice.
- A toggle ignores the password when matching, for finding near-duplicates. Nothing is preselected while it is on.

## How the keeper is chosen

Each entry is ranked by what it carries: a passkey first, then an authenticator code, then number of saved sites, note, favorite, folder, and finally the most recently changed. The top-ranked copy is preselected and the rest are marked for deletion.

Some groups get **no** preselection and an amber warning instead, because an automatic answer would be wrong:

- One copy holds a passkey and a different copy holds the authenticator code.
- More than one copy holds a passkey.
- The copies carry different authenticator secrets.
- A personal entry is grouped with one owned by an organization.

## Limitations worth knowing

- **Passkey detection depends on the export.** The page reads `login.fido2Credentials`. I have not confirmed that every Bitwarden version writes passkey data into a plain JSON export, and passkeys cannot be restored from one. If the field is absent everywhere in your file the page says so — but verify in the app before deleting an entry you believe holds a passkey.
- **Re-importing the cleaned JSON adds entries, it does not replace your vault.** You would end up with everything twice. Treat the cleaned file as a record; do the deleting in Bitwarden.
- **Domain matching is a heuristic.** Registrable domains are derived with a short list of known multi-part suffixes (`co.uk`, `com.au`, and so on) rather than the full Public Suffix List, so an unusual suffix may group more loosely than you expect. Switch to hostname matching if that matters for your vault.
- **Only login items are examined.** Cards, identities, and secure notes are ignored, as are items in the trash.
- **Nothing is deleted for you**, by design. The output is a list you act on yourself or import the updated list to a freshly cleaned vault.

## Browser support

Any current version of Firefox, Chrome, Edge, or Safari. It uses `<dialog>`, `URL.createObjectURL` and CSS custom properties, and follows your system light or dark setting.

## Contributing

Issues and pull requests are welcome. Keep it one file with no dependencies and no network access — that constraint is the point, not an accident. Never attach a real export to an issue; use synthetic data.

## License

[MIT](LICENSE). Provided as is, with no warranty. You are responsible for what you delete from your own vault.

Not affiliated with, endorsed by, or sponsored by Bitwarden, Inc. Bitwarden is a trademark of its owner and is used here only to describe the export format this tool reads.
