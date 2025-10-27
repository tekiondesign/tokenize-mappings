# Tokenize Mappings

Tokenize Mappings is the central configuration and template repository for the Tokenize Figma plugin. It acts as a CMS for designers, letting you map Figma Variables into generated code using predefined templates.

---

## Overview

The Tokenize plugin automates exporting design tokens from Figma into consumable formats such as CSS and JSON. This repository stores template definitions and a central config that the plugin reads dynamically to populate options for designers.

When a user selects a template in the plugin, Tokenize fetches the corresponding `.template` file from this repository and replaces variable references with values from the open Figma file.

---

## Repository Structure

```
tokenize-mappings/
├─ config/
│  └─ templates.json              # Template definitions loaded by the plugin
└─ templates/
   ├─ drp/
   │  ├─ prod/css.template
   │  ├─ stage/css.template
   │  └─ non-prod/css.template
   └─ aec-studio/json.template
```

---

## templates.json schema

The plugin loads `config/templates.json` to populate the Template dropdown. Each entry points to a `.template` file and declares an output type.

Example:

```json
[
  {
    "name": "GM DRP Test",
    "type": "css",
    "path": "https://raw.githubusercontent.com/tekiondesign/tokenize-mappings/refs/heads/main/templates/drp/prod/css.template"
  },
  {
    "name": "GM DRP Stage",
    "type": "css",
    "path": "https://raw.githubusercontent.com/tekiondesign/tokenize-mappings/refs/heads/main/templates/drp/non-prod/css.template"
  },
  {
    "name": "GM DRP Pre-Prod",
    "type": "css",
    "path": "https://raw.githubusercontent.com/tekiondesign/tokenize-mappings/refs/heads/main/templates/drp/stage/css.template"
  },
  {
    "name": "AEC Studio JSON",
    "type": "json",
    "path": "https://raw.githubusercontent.com/tekiondesign/tokenize-mappings/refs/heads/main/templates/aec-studio/json.template"
  }
]
```

Fields:

- `name`: Display name shown in the plugin Template dropdown.
- `type`: Output format. Common values are `css` and `json`.
- `path`: Public raw URL to the template file in this repository.

---

## Template syntax

Templates are plain text files processed by the plugin. You embed lookups to Figma Variables using a helper: `getValue(data, pathArray)`.

Example CSS template:

```css
.test-class-prod {
  height: ${getValue(data, ["tokens.css", "accordion-container-bg", "height"])};
  padding: ${getValue(data, ["tokens.css", "accordion-container-bg", "padding-top"])}px
           ${getValue(data, ["tokens.css", "accordion-container-bg", "padding-right"])}px
           ${getValue(data, ["tokens.css", "accordion-container-bg", "padding-bottom"])}px
           ${getValue(data, ["tokens.css", "accordion-container-bg", "padding-left"])}px;
  border-radius: ${getValue(data, ["tokens.css", "accordion-container-bg", "border-top-left-radius"])}px
                 ${getValue(data, ["tokens.css", "accordion-container-bg", "border-top-right-radius"])}px
                 ${getValue(data, ["tokens.css", "accordion-container-bg", "border-bottom-right-radius"])}px
                 ${getValue(data, ["tokens.css", "accordion-container-bg", "border-bottom-left-radius"])}px;
  border-bottom-width: ${getValue(data, ["tokens.css", "accordion-container-bg", "border-bottom-width"])}px;
}
```

Path format for lookups:

```
["<collection>", "<group>", "<variable>"]
```

Interpretation:

- `<collection>` is the Figma Variable collection. Example: `tokens.css`.
- `<group>` is the variable group within that collection. Example: `accordion-container-bg`.
- `<variable>` is the variable name. Example: `height`.

Practical examples:

| Goal | Path |
| --- | --- |
| Read the color for primary button background | `["tokens.css", "buttons-primary-bg", "color"]` |
| Read heading text size defined in a JSON-oriented collection | `["tokens.json", "text-heading-xl", "font-size"]` |

If a path does not match any variable in the open Figma file, the output will be missing or fall back according to plugin logic. Keep names consistent between Figma and templates.

---

## Using the plugin with this repository

1. Open your Figma file that contains the Variables to export.
2. Launch the Tokenize plugin.
3. From the Template dropdown, select a template defined in `config/templates.json`.
4. Select one or more modes. Modes map to Figma variable modes such as Light and Dark.
5. Click Sync tokens. The plugin fetches the template, resolves all lookups, and generates the output for the selected modes.

Notes:

- The list of templates comes from this repository. When `templates.json` changes, the plugin picks up the new options on the next load.
- The selected template’s `type` determines how the output is formatted. For example, `css` renders CSS, `json` renders structured JSON token data.

---

## Adding a new template

1. Create a `.template` file under `templates/{program}/{environment}/`. Choose any structure that suits your program.  
   Example path: `templates/drp/dev/css.template`.

2. Write your mappings using `getValue(data, [...])` for each property you want to resolve.

3. Add an entry to `config/templates.json`:

```json
{
  "name": "GM DRP Dev",
  "type": "css",
  "path": "https://raw.githubusercontent.com/tekiondesign/tokenize-mappings/main/templates/drp/dev/css.template"
}
```

4. Commit and push. Designers will see the new template in the plugin after reload.

---

## Updating mappings

To change which variable is used for a given property, edit the path arrays in the `.template` file. For example, to switch a padding token to a new group name, update:

```
["tokens.css", "accordion-container-bg", "padding-top"]
```

to

```
["tokens.css", "accordion-container", "padding-top"]
```

Keep Figma variable naming aligned with template paths to avoid missing values.

---

## Conventions and best practices

- Keep template names short and descriptive. Include environment when useful, such as `Prod`, `Stage`, or `Dev`.
- Group templates by program or product line inside `templates/`.
- Prefer consistent collection names in Figma such as `tokens.css` and `tokens.json`.
- When defining padding, border radius, or spacing shorthands, keep units explicit in the template to prevent ambiguity.
- Validate a template against a small test frame before rolling it out to a large file.

---

## Troubleshooting

- Template dropdown is empty: confirm `config/templates.json` is valid JSON and accessible at a public raw URL.
- Output shows `undefined` or blank values: the path does not match a Variable in the open Figma file. Check collection, group, and variable names.
- Wrong values for a mode: verify you selected the expected modes in the plugin and that the Figma Variables have values for those modes.
- Network failures: ensure the `path` URLs in `templates.json` point to the `raw.githubusercontent.com` endpoint.

---

## Roadmap

The `tokenize-mappings` repository is intended to function as a lightweight CMS for designers and engineers. Planned improvements:

- Template metadata such as owner, version, and change notes.
- Validation tools that check template paths against a given Figma file.
- Previews for common output types such as CSS and JSON.
- Program-specific starter templates and lints.

---

## License

Copyright Tekion. Internal use only.
