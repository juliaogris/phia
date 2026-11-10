# phia - code understanding for Go codebases and PRs

phia reads a Go repository, builds one graph of its symbols, and serves
web views of that graph. The first view explains a pull request top
down. A later view explains a whole codebase.

The name is short for Sophia, wisdom. It also refers to Sophie Scholl.

Status: design, 2026-11-10. No Go code exists. Personal project.

- Site: <https://phia.foo>
- Design: [docs/design.md](docs/design.md)

## Problem

Generated code arrives faster than a person can read it. A diff shows
files in alphabetical order. It does not show which change matters, what
the change can affect, or in which order to read it.

## Goal

A reviewer opens a PR in phia and gets three things.

1. A reading order, top down, with a short narrative.
2. The changed symbols, sized by how much they changed.
3. The reach of the change: the callers, implementers and RPCs that the
   change can affect.

Every statement in the narrative links to the code that supports it.

## Non-goals

- Languages other than Go. Proto definitions are read because Go
  services depend on them.
- A replacement for the editor. phia does not edit files.
- A replacement for GitHub review. phia exports notes to a pending
  review. The reviewer submits the review on GitHub.
- A hosted service. phia runs on the reviewer's machine.

## Phases

phia is used on phia from the first day. Each phase has phia itself as
its first user.

| Phase | Builds | phia uses it on | Done when |
|---|---|---|---|
| 0. phia on phia | A static page that shows the design as a graph of sections | the design document | `https://phia.foo` shows `design.md` in three panes |
| 1. Graph engine | Symbol graph at a commit, and the diff of two graphs | the engine's own Go code | `phia diff` lists the changed symbols of a real Teleport PR correctly |
| 2. PR lens | Narrative, changed symbols, reach, notes | every PR to this repository | Julia reviews one real PR in phia and does not open the GitHub diff |
| 3. Codebase lens | Ranked map and guided tours of a whole repository | this repository | A reader new to a package can name its five main types after ten minutes |

Phase 1 serves both lenses. Phase 3 starts after Phase 2 shows which
ranking of symbols is useful.

## Site

The site is Phase 0. `docs/index.html` reads `docs/design.md` in the
browser and draws an outline, a map and the text from it. The page does
not have a build step. To change the site content, edit `design.md`.

Run the site on a local machine:

```bash
python3 -m http.server -d docs 8000
```

### GitHub Pages

GitHub Pages serves the `docs/` directory of the `main` branch.

| File | Purpose |
|---|---|
| `docs/CNAME` | contains the custom domain, `phia.foo` |
| `docs/.nojekyll` | turns off Jekyll, so GitHub serves the files as they are |
| `docs/vendor/` | marked 12.0.2 (MIT), d3-hierarchy 3.1.2 (ISC), mermaid 10.9.1 (MIT) |

Settings, under Settings, Pages, in the repository:

1. Source: "Deploy from a branch", branch `main`, folder `/docs`.
2. Custom domain: `phia.foo`.
3. "Enforce HTTPS": on, after GitHub has issued the certificate.

### DNS at Cloudflare

`phia.foo` is registered at Cloudflare. Add these records under DNS,
Records. Set every record to "DNS only" (grey cloud).

| Type | Name | Content |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `juliaogris.github.io` |

The addresses are the GitHub Pages addresses for an apex domain. They
were written from memory. Compare them with the GitHub Pages
documentation for custom domains before saving.

Three facts about this setup:

- "DNS only" lets GitHub issue the Let's Encrypt certificate. With the
  Cloudflare proxy on (orange cloud), GitHub may fail to issue or renew
  the certificate.
- The `.foo` top-level domain is on the HSTS preload list. Browsers do
  not open `http://phia.foo`. The site does not load until the
  certificate exists. Issuing can take up to an hour.
- Verify the domain under the GitHub account settings, Pages, "Add a
  domain". A verified domain cannot be claimed by another GitHub
  account if this repository stops using it.

Check the records:

```bash
dig +short phia.foo A
```

```bash
dig +short www.phia.foo CNAME
```

## License

MIT. See [LICENSE](LICENSE).
