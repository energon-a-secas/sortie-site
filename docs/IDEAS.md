# Sortie: where this could go

Speculative. Nothing here is built, and some of it should probably stay
unbuilt. Each idea says what you would see, what it needs, and how far it is
from what exists today, because "wouldn't it be cool if" is cheap and the
distance is the part worth knowing.

The through-line: **Sortie already renders 21 independent things, each with a
state, arranged so you take in all of them at once.** That is a status board
that happens to look like a mech. Everything below is a different answer to
"twenty-one of what?"

---

## 1. Each component bound to a live URL

The one you asked about, and the foundation for most of the rest.

**What you would see.** A part is no longer just equipped or empty. It has a
pulse. A healthy component sits at full ink. A degraded one drops to the mid
tone and its trace line to the core flickers. A dead one goes to shadow and the
frame rating falls. The console stops being a picture of a machine and becomes
an instrument reading one.

**The binding.** A part gains an optional block:

```json
{
  "id": "fx-api-gateway",
  "name": "Gateway",
  "slot": "coreL",
  "probe": {
    "url": "https://status.example.com/api/gateway",
    "every": 60,
    "ok": "$.status == 'operational'",
    "label": "$.region"
  }
}
```

`ok` is a tiny expression over the response, not arbitrary code. A part that
cannot be evaluated reports *unknown*, which must look different from *healthy*.
The failure mode of every status board is a green light that means "no data".

**What it needs.** A polling loop with backoff, a three-state model
(ok / degraded / unknown) rather than a boolean, and CORS. That last one is the
real obstacle: most status endpoints will not serve a browser from another
origin. Either the target opts in, or a Worker proxies it, which is the pattern
`radar-site` already uses in this fleet.

**Distance:** a weekend, minus the CORS problem, which is not a weekend.

---

## 2. The Neorgon fleet as the frame

The version that needs almost no new infrastructure, because the data already
exists.

`docs/site-registry.json` describes 64 sites, and `scripts/health-check.sh`
already probes them. Point Sortie at that and every site becomes a component:
the hub is the core, the CDN is the generator, the Convex-backed sites are the
back units, the internal tools are the internals. One glance tells you the
fleet is up. A dead site is a dark tile on a mech.

The frame rating becomes fleet health. Burst modes become views: **Recon**
highlights sites with stale content, **Siege** highlights the ones with
backends to worry about.

**Distance:** short. The registry is a static JSON file this site could fetch
directly, and the health data is already being produced.

---

## 3. A build pipeline you can watch

Each stage of a CI run is a part. The frame assembles as the pipeline
progresses: checkout lights the core, tests light the arms, deploy lights the
legs. A failed stage does not just go red; the tiles downstream of it never
come online, so the shape of the failure is the shape of the mech.

This is the two-wave boot doing real work: the frame wave is the build, the
accessory wave is the deploy.

**Distance:** medium. GitHub Actions has an API; the mapping from jobs to slots
would need to be declared per repo.

---

## 4. The device ID as an actual key

Right now `CORE` is a prop and the README says so. It could be a real
credential without becoming a real lock: a **passkey**. `navigator.credentials`
gives a WebAuthn assertion, the console unlocks on your fingerprint, and the
ceremony gains the one thing it currently fakes.

Worth being clear about what this does and does not buy. It still protects
nothing, because there is still nothing behind it. What changes is that the
gesture becomes true: you really are presenting a device to the socket.

**Distance:** an afternoon. WebAuthn with no server is a discoverable credential
and a local assertion.

---

## 5. Parts that carry provenance

A forged part currently knows its glyph and its stats. It could know where it
came from: who made it, when, what it was forged from. Then the library becomes
a lineage, and a share link carries not just a build but a history.

The interesting version is **remixing**. You import someone's frame, change two
parts, and the export records the fork. Over enough hands you would get a
family tree of frames, which is the thing that makes part-customisation games
last longer than their campaigns.

**Distance:** small technically, large in that it only means anything with more
than one person using it.

---

## 6. A shared hangar

The fleet already runs Convex behind several sites. A backed Sortie could hold
public frames: submit a build, browse other people's, vote. The forge stays
local, the gallery is shared.

The honest caution: this converts a toy with no moving parts into a service with
moderation, spam, and a database that has to stay up. The current version's best
property is that it cannot break, because there is nothing to break.

**Distance:** medium, and it is the idea that most changes what this is.

---

## 7. Ambient telemetry

Bind the *budgets* rather than the parts. EN load reads your machine's actual
CPU, load capacity reads memory pressure, and the frame goes overweight when
your laptop is struggling. The burst modes fire on thresholds. A second monitor
running Sortie tells you the state of the first one at a glance, without a
number anywhere on screen.

**Distance:** far in a browser, which cannot see any of that. Natural if this
were ever an Electron or Tauri shell.

---

## 8. The attract loop as a screensaver with a job

Idle attract already cycles the modes. Give it content: while idle, cycle
through *other people's* frames from the shared hangar, or through the fleet's
sites, or through the day's build history. The console keeps working when
nobody is looking at it, which is the most mech thing it could do.

**Distance:** trivial once any of ideas 1, 2, 3 or 6 exist. It is a renderer,
not a feature.

---

## What ties these together

Three things Sortie would need before most of the above stops being awkward:

1. **A part-source abstraction.** Today a part is a literal in `parts.js` or a
   row in `state.custom`. It wants to be an interface with several
   implementations: stock, forged, fetched, probed.
2. **A three-state health model.** Not a boolean. *Unknown* has to be visually
   distinct from *healthy*, or the console will lie by omission the first time
   a fetch fails.
3. **Somewhere to declare bindings.** A JSON manifest the site loads, so a
   binding is data rather than a code change, and so one deployment can be
   pointed at different things.

None of that is large. It is the difference between a console that draws a mech
and a console that reads something, and it is worth doing before the fourth
idea gets bolted on rather than after.
