# Systems for Frontier Labs

An interactive book on distributed systems, LLM inference infrastructure, and platform engineering, written from first principles for senior backend engineers preparing for infrastructure roles at frontier AI labs.

Every chapter builds on the ones before it. Most chapters end in a lab: a small simulator running in the page that lets you cause the failure the chapter describes, rather than read about it.

**Live site:** `https://<your-user>.github.io/<repo>/` (see [Deploying](#deploying))

## What's inside

The book is one static file, `index.html`. No server, no build step, no dependencies beyond a browser. Fonts load from Google Fonts and fall back to system fonts offline.

| Part | Title | Chapters |
|---|---|---|
| 1 | Distributed systems fundamentals | 0–13 |
| 2 | Messaging, storage, caching, orchestration | 14–21 |
| 3 | System design method and canonical systems | 22–31 |
| 4 | LLM inference infrastructure | 32–38 |
| 5 | Kubernetes, cloud, and infrastructure as code | 39–46 |
| 6 | Production operations and reliability | 47–51 |
| 7 | Data infrastructure | 52–56 |
| 8 | Low-level systems | 57–63 |
| 9 | Security and privacy engineering | 64–65 |
| 10 | Staff-level engineering practice | 66–67 |
| 11 | Domain context: the labs | 68–69 |
| 12 | Capstone build | 70–73 |

Each chapter is sized for a two-hour live session: roughly thirty minutes of concept, an hour with the lab, twenty minutes on failure modes, and homework.

## Status

| Chapter | Title | Text | Lab |
|---|---|---|---|
| 0 | Why distributed systems are hard | ✅ | Client/server over a lossy, duplicating network; idempotent-server switch |
| 1 | Consistency models, made concrete | ✅ | Replicated register with stale and non-monotonic read detection |
| 2 | CAP, PACELC, and what "AP" costs | ✅ | Partition switch; CP vs AP with conflict accounting |
| 3 | Replication architectures | ✅ | Leader failover; lost acknowledged writes under async vs semi-sync |
| 4 | Leaderless systems: the Dynamo toolkit | ✅ | Quorum explorer with N/R/W, churn, hinted handoff, read repair |
| 5 | Conflict resolution and CRDTs | ✅ | Two-replica PN-Counter and OR-Set with visible internals |
| 6 | Raft from scratch: election and log replication | ✅ | Five-node Raft: elections, heartbeats, commits, crashes, partitions |
| 7–73 | — | outline only | planned |

Scaffolded chapters have final learning objectives, prerequisites, and a session outline. They are marked in the contents rail and on the page.

## Running locally

Open `index.html` in a browser. That's it.

If you want a local server (some browsers restrict `file://` for fonts):

```
python3 -m http.server 8000
# then http://localhost:8000
```

## Deploying

The site is a single static file, so any static host works.

**GitHub Pages**

1. Settings → Pages → Source: *Deploy from a branch*, branch `main`, folder `/ (root)`.
2. The site is served at `https://<your-user>.github.io/<repo>/`.

Pushes to `main` redeploy automatically. Netlify, Cloudflare Pages, and Vercel work the same way with no build command and `/` as the output directory.

## How the book is structured

Everything lives in `index.html`, in four blocks:

1. **CSS** — design tokens and layout. One serif for reading, one sans for interface, mono for code and event logs. The single accent colour is reserved for interactive state; amber and green are semantic colours used only inside labs.
2. **Content** — a `CH` object mapping chapter ids (`s0` … `s73`) to chapter records, plus a `STUBS` array for scaffolded chapters and a `PARTS` list.
3. **Widgets** — a `WIDGETS` object mapping widget names to functions that render a lab into a container.
4. **App** — table of contents, hash routing, chapter rendering, prerequisite links, in-memory progress.

### Chapter record

```js
CH.s9 = {
  n: 9,                 // chapter number, used for ordering and cross-references
  part: 3,              // part id (see PARTS)
  title: "Idempotency and exactly-once",
  widget: "idempotency",// key in WIDGETS, optional
  on: ["s8", "s3"],     // prerequisite chapter ids; rendered as "Builds on" links
  obj: [ ... ],         // learning objectives
  html: ` ... `,        // chapter body as HTML inside a template literal
  hw: [ ... ]           // homework items
};
```

"Used by" links are derived automatically from other chapters' `on` arrays.

### Widget contract

A widget is a function `(root) => void` that renders into `root` and may start a timer:

```js
WIDGETS.example = function(root){
  root.innerHTML = labShell(title, hint, controlsHTML, stageHTML, single);
  root._t0 = performance.now();      // for the event log clock
  const log = mkLog(root);           // log(cls, msg); cls ∈ ok | warn | bad | info | t
  // ... state, controls, draw() ...
  root._timer = setInterval(tick, 60); // the app clears this on navigation
};
```

`labShell` builds the standard bench: header, controls row, stage on the left, event log on the right (`single = true` omits the log). Use CSS classes `.node`, `.node.leader`, `.node.dead`, `.node.partitioned`, `.msg`, `.msg.stale`, `.lbl`, `.val` inside stage SVGs so labs look consistent.

Two rules learned the hard way:

- Never mutate the message array from inside a `filter` callback that will reassign it; collect due messages first, then deliver.
- Don't use `localStorage` or `sessionStorage`. Progress is intentionally in memory only.

### Adding a chapter

1. Replace the entry in `STUBS` with a full `CH.sN = { ... }` record (keep `n`, `part`, `on`, and `obj`).
2. Write the body in the same shape as existing chapters: motivate from the previous chapter, define by anomaly or mechanism, give the failure scenarios, end with a "Lab:" section that explains what the simulator shows.
3. Add the widget under `WIDGETS` and reference it by name.
4. Run the syntax check and a smoke test before committing.

### Checking your changes

```
# syntax check the script block
python3 -c "s=open('index.html').read();open('/tmp/b.js','w').write(s.split('<script>')[1].split('</script>')[0])"
node --check /tmp/b.js
```

A headless smoke test with Playwright (navigate to each chapter, click through each lab, assert no page errors) lives in the workflow history and is worth adding as `test/smoke.py` if you take contributions.

## Roadmap

- Part 1 completion: chapters 7–13 (Raft safety and membership, leases and fencing built on the Raft lab, idempotency and the outbox, consistent hashing, hybrid logical clocks, the retry-storm simulator, gossip and cells).
- Part 4 (inference) next, since it is what the platform roles hire for.
- Optional split into `index.html` + one file per chapter for smaller diffs once the chapter count justifies it.

## Sources and further reading

Designing Data-Intensive Applications (Kleppmann); the Dynamo paper (DeCandia et al., 2007); In Search of an Understandable Consensus Algorithm (Ongaro & Ousterhout, 2014); Jepsen analyses; the vLLM and SGLang papers and source; Kubernetes source (scheduler, controller-runtime); Site Reliability Engineering (Google).

## License

Text: CC BY-NC-SA 4.0. Code (simulators and app shell): MIT.
