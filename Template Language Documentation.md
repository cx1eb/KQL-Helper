# Template Language Documentation

## 1. Syntax Overview

The system supports two main kinds of tags:

* **Variable placeholders**

  ```
  {{ variable_name }}
  {{ variable_name type }}
  ```

  These are replaced with values from the schema/context at render time.

  * Supports optional type annotations (`string.nullable`, `integer`, `datetime`, `identifier`, `array.string`, `enum:[...]`).

* **Conditional blocks**

  ```
  {% if variable_name %}
    ... lines of query ...
  {% endif %}
  ```

  These include or omit blocks based on the truthiness of the variable.

* **Comments**

  * Any line starting with `//` is ignored when scanning for variables.

---

## 2. Supported Types

When defining schema or annotating a variable in a template, the following types are recognized:

| Type              | Default Handling                                                               |
| ----------------- | ------------------------------------------------------------------------------ |
| `string`          | Defaults to empty string `""`.                                                 |
| `string.nullable` | Defaults to `null`. Renders empty when null.                                   |
| `integer`         | Defaults to `0`.                                                               |
| `datetime`        | Defaults to current ISO8601 string (`YYYY-MM-DDTHH:MM:SSZ`).                   |
| `identifier`      | Used for KQL identifiers (e.g., table/column names).                           |
| `array.string`    | Defaults to an empty array (`[]`). Rendered as a JSON array string.            |
| `enum:[...]`      | Treated as a dropdown list. First entry is default unless otherwise specified. |

---

## 3. Rendering Behavior

* **Variable resolution**
  `{{ var }}` inserts the current value.

  * Arrays are JSON-encoded (e.g., `["1.2.3.4","5.6.7.8"]`).
  * `null`/`undefined` values render as empty.

* **Conditionals**
  Truthiness is defined as not `undefined`, not `null`, not empty string, not `0`, not `false`.

* **Schema Auto-Discovery**

  * `scanVars()` detects all `{{ var type? }}` and `{% if var %}` references in a template.
  * Missing variables are auto-added to the schema with defaults (via `ensureSchemaCoversTemplate()`).

---

## 4. Example Template

```kql
// Example query
let StartTime = datetime({{ time_start datetime }});
let EndTime   = datetime({{ time_end datetime }});
let TargetIPs = dynamic({{ ip_list array.string }});
{{ table_name identifier }}
| where TimeGenerated between (StartTime .. EndTime)
| where IPAddress in (TargetIPs)
{% if username string.nullable %}
| where UserPrincipalName == '{{ username }}'
{% endif %}
| summarize count() by IPAddress, bin(TimeGenerated, {{ bin_minutes integer }}m)
| order by count_ desc
```

### Example Rendered Output

(with schema values filled in)

```kql
let StartTime = datetime(2025-01-01T00:00:00Z);
let EndTime   = datetime(2025-12-31T23:59:59Z);
let TargetIPs = dynamic(["1.2.3.4","5.6.7.8"]);
SigninLogs
| where TimeGenerated between (StartTime .. EndTime)
| where IPAddress in (TargetIPs)
| where UserPrincipalName == 'alice@example.com'
| summarize count() by IPAddress, bin(TimeGenerated, 15m)
| order by count_ desc
```

---

## 5. Schema JSON Format

Each query has an associated schema (JSON object). Example:

```json
{
  "time_start": {"type": "datetime", "default": "2025-01-01T00:00:00Z"},
  "time_end": {"type": "datetime", "default": "2025-12-31T23:59:59Z"},
  "table_name": {"type": "identifier", "default": "SigninLogs"},
  "ip_list": {"type": "array.string", "default": ["1.2.3.4","5.6.7.8"]},
  "username": {"type": "string.nullable", "default": null},
  "bin_minutes": {"type": "integer", "default": 15, "min": 1, "max": 240}
}
```

---

## 6. Key Functions in Engine

* `renderTemplate(tpl, ctx)` → renders template with context.
* `scanVars(tpl)` → finds all `{{ var type }}` and `{% if var %}`.
* `normalizeSchema(schema, tpl)` → ensures schema matches template.
* `ensureSchemaCoversTemplate()` → adds missing vars into schema automatically.
* `paramContext()` → extracts live values from UI to context.

