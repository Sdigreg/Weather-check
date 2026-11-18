# Perth Morning Ride Check

A small weather helper for Perth, WA. It looks at the forecast for the next few
mornings and tells you whether to **ride**, **walk**, or **stay home for a coffee**.

There are three versions of the same idea. Each one keeps a copy of the last good
forecast, so a bad connection doesn't leave you without an answer.

| Version | Where it is | What it covers |
| --- | --- | --- |
| **Main web page** | <https://sdigreg.github.io/Weather-check/> | Next 7 mornings, 6:00–10:00 AM. Ride / walk / café. |
| **Commute page** | <https://sdigreg.github.io/Weather-check/commute.html> | Next 3 days. Ride to work (6:00–8:00 AM) and ride home (5:00–7:00 PM). |
| **Windows script** | `morning-ride-check.ps1` on your PC | The same 7 mornings as the main page, in a terminal window. |

### Do I have to type `index.html`?

**No.** GitHub Pages automatically opens `index.html` when you visit the folder
address, so the short address is enough:

- Works: <https://sdigreg.github.io/Weather-check/>
- Also works: <https://sdigreg.github.io/Weather-check/index.html>

Use the short one. The commute page *does* need its file name in the address,
because it is not the "default" page of that folder.

---

## How the verdict works

The main page and the Windows script use these rules:

| Verdict | Peak chance of rain | Average wind |
| --- | --- | --- |
| 🚴 **Ride** | 20% or less | 25 km/h or less |
| 🚶 **Walk** | 35% or less | 30 km/h or less |
| ☕ **Café** | Anything worse | Anything worse |

The commute page uses the ride rule only: ride if the chance of rain is 20% or
less **and** the wind is 25 km/h or less. Otherwise, drive.

---

## When the forecast won't load

Weather services have bad days too. So every time one of the three versions runs
successfully, it quietly saves the weather numbers it just received, plus the
date and time it saved them.

If the live forecast cannot be reached, you still get an answer:

1. It waits **30 seconds** for an answer.
2. If nothing comes back, it tries again — **three tries in total**.
3. Only after **all three** tries fail does it show the saved copy.

The saved copy appears with a note in this style:

> **Archived results:** Last ran on 10th July 2026 - 4:35PM AWST

Underneath that note, the normal ride / walk / café logic (or the commute logic)
is worked out from the saved numbers, exactly as if they were fresh. The note also
tells you which page or script saved them, so you know where they came from.

**It will never fall back on the first or second failure** — those are treated as
a hiccup and retried. You only see saved results after three failures in a row,
which takes about 90 seconds.

### Where the saved copy lives

| Version | Saved in |
| --- | --- |
| Main web page | Your browser's own storage on that device (`localStorage`), under the key `perth-morning-weather-archive-v1` |
| Commute page | The **same key**, so a good run on either page can rescue the other |
| Windows script | `morning-ride-check-archive.json`, in the same folder as the script |

Both web pages sit on the same website (`sdigreg.github.io`), so they share that
storage automatically — on the same device and browser. The Windows script can't
see your browser's storage, which is why it keeps its own file.

> **Important:** the storage key must stay identical in both pages:
> `perth-morning-weather-archive-v1`. If you change it in one page, change it in
> the other too, or they will stop helping each other.

---

## One shared archive for all three (optional, about 2 minutes)

The pages and the script can also copy the saved numbers to a small **free
online store**, so all three use the same latest copy — even on different devices.

### Can GitHub do this on its own?

No, not safely. GitHub Pages only *serves* files; it is a display cabinet, not a
filing cabinet. Writing a file back to GitHub needs a secret access key, and
anything inside a public web page can be read by anyone who looks. A leaked key
could let a stranger change your files. So GitHub can't safely be the shared
filing cabinet for a public page — we use a small free storage service instead.

### What to use instead

[Pantry](https://getpantry.cloud) is a free service made for exactly this: store a
small piece of data and fetch it again later. It needs no account, no email and
no payment. The free plan is far bigger than we need — our file is only a few
kilobytes.

**Set it up once:**

1. Open <https://getpantry.cloud> and click **Create a Pantry**.
2. It shows you a long ID (like `a1b2c3d4-...`). Copy it.
3. Your storage address is the same for reading and writing:
   `https://getpantry.cloud/apiv1/pantry/PASTE-YOUR-ID-HERE/basket/weather`
4. Paste that address into **all three** files, as shown below.
5. Run any one of them once while online. That uploads the first copy.

**In `index.html` and `commute.html`**, find this block near the top of the script
and make it look like this (keeping your own ID):

```js
const CLOUD_ARCHIVE = {
  enabled: true,
  readUrl:  "https://getpantry.cloud/apiv1/pantry/PASTE-YOUR-ID-HERE/basket/weather",
  writeUrl: "https://getpantry.cloud/apiv1/pantry/PASTE-YOUR-ID-HERE/basket/weather",
  timeoutMs: 15000
};
```

**In `morning-ride-check.ps1`**, find these three lines near the top:

```powershell
$CloudArchiveEnabled = $true
$CloudArchiveReadUrl  = "https://getpantry.cloud/apiv1/pantry/PASTE-YOUR-ID-HERE/basket/weather"
$CloudArchiveWriteUrl = "https://getpantry.cloud/apiv1/pantry/PASTE-YOUR-ID-HERE/basket/weather"
```

That's it. From then on:

- every successful run saves to the shared store as well as locally;
- if a page or the script fails three times, it uses the **newest** copy it can
  find — the local one or the shared one, whichever was saved last;
- if the store is unreachable, nothing breaks. The local copy still works.

**To check it worked:** open either page and look at the small print at the bottom.
It should say "shared archive updated". If it says "shared archive not updated",
the address or the ID is not quite right — the page itself still works.

### A few honest notes

- **This is optional.** Without it, the two web pages still share saved results
  with each other in the same browser, and the script keeps its own file.
- **The address is not a password.** Anyone who finds that link could read or
  overwrite the saved weather numbers. That is fine for weather — just don't put
  anything private in there.
- **It is a small free service.** If it ever goes away, the pages and script carry
  on using their own local copies. To switch, paste a new address in the same
  three places (any service that accepts a `GET` to read and a `PUT` to write
  works).
- **Old data may be cleared.** Free services sometimes tidy up storage that has
  not been touched for a long time. That's harmless here: the next good run
  simply saves a fresh copy.
- **To turn it off again:** set `enabled` to `false` in the pages, or
  `$CloudArchiveEnabled = $false` in the script.

---

## The files

| File | What it is |
| --- | --- |
| `index.html` | The main web page (ride / walk / café). |
| `commute.html` | The commute page for your friend. |
| `morning-ride-check.ps1` | The Windows script. |
| `morning-ride-check.csv` | The cycling affirmations used by the script. Unchanged. |
| `morning-ride-check-archive.json` | Created automatically by the script the first time it succeeds. |
| `README.md` | This file. |

Both web pages are single files on purpose: each one holds its own styles,
settings and code, so there is nothing extra to upload or keep in sync.

---

## Changing the settings

Everything you might want to adjust is near the top of each file.

**In `index.html`:**

| Setting | What it does |
| --- | --- |
| `LAT`, `LON` | The location being forecast |
| `CYCLE_RAIN_MAX`, `CYCLE_WIND_MAX` | The riding limits |
| `WALK_RAIN_MAX`, `WALK_WIND_MAX` | The walking limits |
| `REQUEST_TIMEOUT_MS` | How long one try may take (30 seconds) |
| `MAX_ATTEMPTS` | How many tries before saved results appear (3) |
| `ARCHIVE_KEY` | The shared storage name — keep it identical in both pages |
| `ICONS` | What is shown for each verdict (emoji now, pictures later) |

**In `commute.html`:** `LAT`, `LON`, `RAIN_MAX`, `WIND_MAX`, the `WINDOWS` list
(the time slots), and the same timeout / attempt / archive settings.

**In `morning-ride-check.ps1`:** `$ApiTimeoutSeconds` (30), `$MaxAttempts` (3),
the three cloud settings, and the four rain/wind threshold numbers.

---

## Updating the live pages

1. Open your repository on GitHub (`sdigreg/Weather-check`).
2. Open `index.html` (or `commute.html`), click the pencil button, select
   everything, and paste in the new file. Commit the change.
3. Wait a minute for GitHub to publish it.
4. Open the page and press **Ctrl + F5** (Windows) to force a fresh copy. Without
   that, your browser may show the old version from its cache.

The file names and folder stay exactly the same, so the addresses don't change.

---

## Troubleshooting

**I see the "Archived results" note.**
That is the fallback working. The live forecast could not be reached three times
in a row. It will go back to normal on the next successful run.

**I see "no saved results to fall back on yet".**
The forecast failed and this is the very first time, so there is nothing saved.
Check the internet and try again.

**The page is stuck on "Loading forecast…" for a while.**
That is normal on a slow connection: it may wait up to 30 seconds per try, three
times. You will see "Trying again (2 of 3)…" between tries.

**The shared archive isn't updating.**
Check that `enabled` is `true` and that the address was pasted into **both**
`readUrl` and `writeUrl`, in every file. The page footer says "shared archive
updated" or "shared archive not updated".

**Private browsing.**
Some browsers block or forget storage in private windows. The page still works,
but the saved copy may not survive.

**The script says it couldn't save.**
The folder holding the script is not writable. Move the script somewhere in your
own user folder, or run it from there.

---

## Changelog

### 2.0 — 18 November 2026

- **30-second timeout everywhere.** The Windows script went from 15 to 30 seconds
  per try. The two web pages previously had no time limit at all; they now give
  up after 30 seconds, so a dead connection can't leave them spinning.
- **Saved results.** Every successful run now stores the forecast and a timestamp.
  After three failed tries, the last good numbers are shown with an
  "Archived results: Last ran on …" note. It never triggers on the first or
  second failure.
- **One shared key for both pages.** `index.html` and `commute.html` read and
  write the same browser storage entry, so either page can rescue the other.
- **Optional shared cloud archive.** A free online store can hold one copy for
  the two pages and the Windows script alike. Off by default; see the setup
  section above.
- **Clearer status messages.** "Trying again (2 of 3)…" while retrying, and a
  plain explanation when there is nothing saved yet.
- **README rewritten** with the real live links, a note about `index.html`, and
  this changelog.

### 1.0 — earlier

- The original version: 7 mornings of ride / walk / café advice from Open-Meteo,
  a 15-second connection timeout, cycling affirmations, and a commute page.

---

## Where the weather comes from

Forecast data comes from [Open-Meteo](https://open-meteo.com/), which is free for
personal, non-commercial use and needs no sign-up or key. The pages fetch the
numbers directly in your browser; there is no server of ours in between.

Perth is always 8 hours ahead of UTC (AWST) and has no daylight saving, so the
"AWST" times in the saved-results note are always Perth local time.
