# Layer 2 - HaramBlur (content blurring on the page)

Pi-hole is a **network** filter: it blocks bad *domains* and forces SafeSearch for
every device, but it cannot look *inside* a page it allows. **HaramBlur** is the
second layer that covers that gap - a free, open-source browser extension that uses
**on-device AI** to automatically **blur inappropriate images and videos on the page
itself**, in real time.

> HaramBlur is a **separate project**, not part of this kit and not affiliated with it.
> Source & credit: **<https://github.com/alganzory/HaramBlur>** (license: **AGPL-3.0**).
> It uses face detection (the *Human* library) + NSFW detection (*nsfwjs*).

## Why use it *with* Pi-hole

| | Pi-hole (Layer 1) | HaramBlur (Layer 2) |
|---|---|---|
| Works at | The **network** (DNS), on the Pi | The **browser**, on each device |
| Blocks / hides | Bad **domains** + forces SafeSearch/YouTube | Bad **images & video** on allowed pages |
| Covers | Every device on the Wi-Fi, no per-device app | Only browsers where it's installed |
| Blind spot it has | Can't see *inside* an allowed page | Only the browser (not apps/other devices) |

Neither is perfect alone. Together they cover far more: the network gate stops most
bad sites; the on-page filter blurs what slips through on the sites that remain.

## Install (per browser, per device)

Install it in **each family member's browser**:

- **Chrome / Edge / Brave (Chromium):**
  <https://chrome.google.com/webstore/detail/haramblur/pbcoegikffnadpahojjhgdladmmddeji>
- **Firefox (desktop & Android):**
  <https://addons.mozilla.org/addon/haramblur/>

After installing, open the extension's pop-up and set:
- **Detection type** - images/video, faces, NSFW.
- **Blur strength** and **strictness** - start moderate; raise for younger kids.
- **Hover-to-unblur** - consider turning this **off** on children's devices.
- **On/off toggle** - leave it on.

## Good to know (honest notes)

- **Per-device, per-browser.** Unlike Pi-hole, it isn't network-wide - you install it
  on each device/browser, and it only protects the browser (not native apps or games).
- **Runs on the device.** All processing is in the browser (private - nothing uploaded),
  but the AI uses CPU/GPU, so it can slow older phones/laptops. Lower the strictness or
  pause it on weak devices.
- **Not foolproof.** AI detection misses things and occasionally over-blurs. It's a strong
  aid, not a guarantee - keep it paired with Pi-hole, device parental controls, and
  conversations.

## Can I enforce it from the network? (the honest answer)

**No - and it's worth understanding why.** HaramBlur is a **browser** extension: its AI runs
*inside* each browser, on each device. Pi-hole (and your router) work at the **DNS** layer -
they only see *domain names*. Once a site is allowed, the page is encrypted end to end, so the
Pi **cannot** look inside it, **cannot** install anything on a device, and **cannot** make a
browser blur an image. That separation is the whole reason HaramBlur exists as a second,
on-device layer - it does the job the network physically can't.

What you **can** do is make it **non-optional on the devices you manage**: *force-install* it
with a browser policy so a child cannot remove or disable it. That's set on the device (once
per device, or centrally through an MDM), not on the Pi.

### What's realistic on each platform

| Platform / browser | Can run HaramBlur? | Can you lock it on (force-install)? |
|---|---|---|
| Windows / macOS / Linux - Chrome, Edge, Brave | Yes | Yes - `ExtensionInstallForcelist` policy |
| Windows / macOS / Linux - Firefox | Yes | Yes - `policies.json` |
| Android - **Firefox** | Yes (Firefox 120+) | Yes, via Firefox enterprise policy / MDM |
| Android - Chrome | **No** (Chrome on Android has no extensions) | Use Firefox instead |
| iPhone / iPad - Safari | **No** (not available) | Use Screen Time + a filtering browser |

### Chrome / Edge / Brave - force-install (locked, non-removable)

Verified extension ID: **`pbcoegikffnadpahojjhgdladmmddeji`** (HaramBlur on the Chrome Web
Store). Confirm it on the store page before you lock it in - the ID is the 32-letter string at
the end of the store URL.

**Windows (Group Policy / registry):**

```
Key:   HKLM\SOFTWARE\Policies\Google\Chrome\ExtensionInstallForcelist
Value: 1  =  pbcoegikffnadpahojjhgdladmmddeji;https://clients2.google.com/service/update2/crx
```

Use `...\Microsoft\Edge\...` for Edge; the same Chrome key works for Brave on most builds.

**Linux (a family laptop):** create `/etc/opt/chrome/policies/managed/haramblur.json`:

```json
{ "ExtensionInstallForcelist": ["pbcoegikffnadpahojjhgdladmmddeji;https://clients2.google.com/service/update2/crx"] }
```

**macOS:** ship the same `ExtensionInstallForcelist` key in a configuration profile (MDM).

The extension then installs silently on next launch, and the user can't turn it off or remove it.

### Firefox (desktop & Android) - force-install

Verified add-on ID: **`info@haramblur.com`** (slug `haramblur`). Put a `policies.json` in
Firefox's policies folder (`/etc/firefox/policies/` on Linux,
`Firefox.app/Contents/Resources/distribution/` on macOS, `<install>\distribution\` on Windows):

```json
{
  "policies": {
    "ExtensionSettings": {
      "info@haramblur.com": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/haramblur/latest.xpi"
      }
    }
  }
}
```

`force_installed` makes it non-removable. On Firefox for Android, apply the same policy through
your MDM.

### Simpler, no-policy option

If editing policies is too much, use the device's **supervised account** to control extensions
and lock the browser choice: **Google Family Link**, **Apple Screen Time**, or **Microsoft
Family**. These are device-level controls outside this kit, but they're what make Layer 2
actually *stick* - without them a determined child can just disable the add-on.
