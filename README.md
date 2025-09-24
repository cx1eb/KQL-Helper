# KQL Configurator (Web)

KQL Configurator is a browser based tool for building, editing, and managing parameterized KQL (Kusto Query Language) templates. It provides a schema driven interface with type aware inputs, live rendering, and secure export options. The application runs entirely in the browser and stores queries locally.

## Features

- Build and edit KQL templates with `{{ variable }}` placeholders and `{% if %}` conditionals.
- Auto generate schema from templates, including type aware inputs (string, integer, datetime, array, enum, identifier).
- Dark themed interface with customizable settings.
- Import and export queries in plain, JSON package, or encrypted formats.
- Supports AES-256-CBC, XTEA-CBC, and RSA hybrid encryption for export.
- RSA 4096-bit key generation, storage, and export directly in the browser.
- LocalStorage persistence for queries and settings.

## Getting Started

1. Clone or download this repository.
2. Open `index.html` in a modern browser (Chrome, Edge, or Firefox).
3. Use the interface to create or load queries, edit templates, and manage schema.
4. Save queries locally or export them for sharing.

## Usage

- **Queries Panel**: Create, search, and manage saved queries.
- **Parameters Panel**: Edit schema driven inputs (auto generated or manually defined).
- **Editor**: Write KQL templates with placeholders. Switch between Template and Rendered views.
- **Export/Import**: Save queries as JSON, encrypted packages, or rendered KQL text.

## Security

- All operations occur in the browser. No data is sent to external servers.
- Encryption options (AES, XTEA, RSA hybrid) are available for secure exports.
- RSA key pairs are generated and stored locally.
