---
title : "Multilingual and i18n"
weight : 25
---

This template is fully compatible with multilingual mode.

## Basic configuration

Define your languages in your `contentspec.yaml` file.

For example with current French and English website.

```yaml
version: 1.0
languageCode: en-US
# English is the default language
defaultContentLanguage: en
languages:
  en:
    languageName: English
    title: "Documentation for Learn Theme"
    weight: 1
  fr:
    languageName: "Français"
    title: "Documentation du thème Learn"
    weight: 1

```

Then, for each new page, append the *id* of the language to the file.

- Single file `index.en.md` is split in two files:
    - in English: `index.en.md`
    - in French: `index.fr.md`