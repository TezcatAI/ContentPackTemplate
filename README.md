# Tezcat Content Pack Template

A starting point for building your own [Tezcat](https://github.com/TezcatAI/Tezcat) content pack. Clone it, change the namespace, replace the example content, and add the repository as a source in the Tezcat Extensions hub.

A *content pack* is a git repository with a `tezcat-pack.yaml` manifest plus one declarative YAML file per content item. Tezcat's package pipeline fetches, validates, and applies a pack into a running instance. This template ships one working example of each content type, wired together so the cross-references resolve; the official first-party pack ([`TezcatAI/ContentPack`](https://github.com/TezcatAI/ContentPack)) is a fuller real-world example.

## Layout

```
tezcat-pack.yaml      # the manifest (namespace, version, contents list)
frameworks/*.yaml     # one risk framework per file (a taxonomy of items)
capabilities/*.yaml   # one typed capability per file, with framework mappings
prompts/*.yaml        # one capability-probe prompt per file
profiles/*.yaml       # one probe profile (a named set of prompts) per file
```

Every item is stamped into your pack's `namespace` on install. You do not have to use all four content types, keep only the directories you need and trim the matching entries from `contents` in the manifest.

## The four content types

The types depend on each other, which is why the manifest lists them in dependency order (frameworks first, profiles last):

- **Frameworks** are risk taxonomies (a list of coded items). Most packs do not ship one, the official `tezcat` pack already carries OWASP ASI, NIST AI RMF, and AVID, and you can reference their item codes directly. Ship a custom framework only for an in-house taxonomy of your own.
- **Capabilities** are the typed behaviours you want to detect in an agent (execute code, read the filesystem, and so on). Each has a kebab-case slug, a default severity, optional `prerequisites`, and `framework_items` mapping it to framework codes.
- **Prompts** are the probes: the text sent to a target agent plus the parser rules that turn the reply into a per-capability present/absent finding. A prompt targets one or more capabilities by slug. The template shows the baseline *direct* self-report style and describes the harder *behavioral* style that asks the agent to actually exercise the capability.
- **Profiles** are named, runnable sets of prompts, selected either dynamically by tag (`filter_spec`) or as a pinned, ordered list (`explicit_list`).

Each example file is commented field-by-field, read them in dependency order (`frameworks` → `capabilities` → `prompts` → `profiles`) to see how the pieces connect.

You rarely need all four. The most common pack is just new **prompts and profiles**: fresh probes that target capabilities Tezcat already tracks and map to frameworks it already carries (MITRE ATLAS, OWASP LLM, OWASP ASI, NIST AI RMF, AVID). Only add a **capability** when you want to detect a behaviour Tezcat does not yet model, and a **framework** only when you have an in-house taxonomy of your own or want to incorporate a new framework. Keep the directories and `contents` entries for the types you use, and delete the rest.

## Getting started

1. Update `tezcat-pack.yaml`: set your own `namespace` (short, lowercase, unique, and not the reserved `tezcat`), `name`, `author`, `homepage`, and `upstream`.
2. Replace the `example-*` files with your own content, keeping the slugs, `target_capability_types`, `framework_items`, and profile references consistent so they resolve against each other.
3. Delete any content type you are not using, both its directory and its entry in the manifest `contents` list.
4. Commit and push to your repository.
5. In the Tezcat Extensions hub, add your repository as a source, then install the pack.

## Versioning and releasing updates

### Version fields

Every content pack has two levels of versioning:

**Pack version** (`version` in `tezcat-pack.yaml`) tracks the pack as a whole. Tezcat displays it in the Extensions hub and uses it in the update-available prompt.

**Content version** (`version` inside each `frameworks/*.yaml`, `capabilities/*.yaml`, `prompts/*.yaml`, `profiles/*.yaml`) forms the content coordinate `namespace:slug@version`. Changing this field creates a new logical item; the old version stays in the database so existing probe-run records remain valid.

Use [semantic versioning](https://semver.org) for both:

| Change | Pack version bump | Content version bump |
|---|---|---|
| Fix a typo or regex in an existing item | patch (`1.0.0 → 1.0.1`) | patch (`1.0.0 → 1.0.1`) |
| Add new items | minor (`1.0.0 → 1.1.0`) | — (new files start at `1.0.0`) |
| Rename slugs, remove items, or change the namespace | major (`1.0.0 → 2.0.0`) | major if slug kept |

### Release workflow

1. Edit content files under `frameworks/`, `capabilities/`, `prompts/`, or `profiles/`.
2. Bump the `version` field in any changed content file.
3. Bump `version` in `tezcat-pack.yaml` to reflect the scope of the change.
4. Commit and push to the repository.
5. In the Tezcat Extensions hub, click **Check** on the pack to detect the new version, then **Update** to apply it. All update types (patch, minor, major) require confirmation before applying.
