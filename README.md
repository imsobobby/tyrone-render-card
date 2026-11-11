# Tyrone 3D Render Card

A self-contained Lovelace card for Home Assistant that puts Tyrone's 3D
house render on a dashboard — and wires it to the real house.

The whole render (a Three.js page built from the house's Sweet Home 3D
layout) is **embedded inside `tyrone-render-card.js`** as base64, together
with the Home Assistant bridge from the original `render-bridge-card`.
There is no render file to host and no URL to maintain: the card decodes
its bundled copy of the render into a `blob:` URL, loads it in an iframe,
and talks to it over `postMessage`:

- **HA → render:** live per-device state (on/off, brightness, colour,
  media/app state, availability) for 28 mapped devices, the mail flag,
  the Party Mode flag, every automation's on/off state keyed by friendly
  name, plus panel data (automation counts, lowest battery %, offline
  entity count).
- **Render → HA:** taps in the render become service calls — lights
  (power, brightness, colour), media players (play/pause/power, app picks
  where the entity exposes a source list), automation toggles, the mail
  switch (`switch.mail_waiting`), and the Party button (presses
  `input_button.party_start` to start; turns `input_boolean.party_mode`
  off to stop).

The Shark IQ in the render is deliberately **not** mapped — it stays a
picture-only device, exactly as in the original bridge card.

## Install

1. In Home Assistant, open **HACS**.
2. Open the **⋮** menu (top right) → **Custom repositories**.
3. Paste this repository's URL, choose category **Plugin** (Lovelace),
   and click **Add**.
4. Find **Tyrone 3D Render Card** in HACS and click **Download**. HACS
   registers `tyrone-render-card.js` as a dashboard resource for you.
5. Add the card to a dashboard (see config below). A full-width view
   (panel view) suits it best.

Requires Home Assistant **2024.6.0** or newer.

## Config

Bundled render (the normal way — no URL needed):

```yaml
type: custom:tyrone-render-card
```

With an explicit height (default is `600px`):

```yaml
type: custom:tyrone-render-card
height: 52dvh
```

The old element name works too — dashboards built with the original
bridge card carry on unchanged:

```yaml
type: custom:render-bridge-card
```

### Legacy URL mode

If a `url` other than `"bundled"` is given, the card behaves exactly
like the original `render-bridge-card`: it iframes that hosted render
page and pins bridge messages to that page's origin.

```yaml
type: custom:tyrone-render-card
url: https://example.org/my-render.html
```

`url: "bundled"` (or no `url` at all) selects the embedded render.

## Notes

- **The render is a fixed snapshot of the house layout.** The 3D model
  inside the card reflects the home as it was when this package was
  built. Moving furniture or renaming devices in Home Assistant does not
  update the picture — a new build of this package does. Device *state*
  is always live; it's the *geometry* that's a snapshot.
- **One file, on purpose.** The single ~950 KB `tyrone-render-card.js`
  is the entire plugin: card, bridge, and render. Updates arrive by
  downloading the new version of that one file (via HACS).
- If the embedded render ever fails to load and no `url` is configured,
  the card shows a short note instead of a broken frame.
