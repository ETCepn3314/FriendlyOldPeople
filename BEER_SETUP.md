# 🍻 Drinks counter — shared setup (one time, ~5 min)

The dashboard's **Drinks** tab (🍺 Beers / 🥃 Shots / 🍷 Winos) works right away,
but until you do this it only saves counts **on each phone**. Follow these steps
once to make the counts **shared** so everyone sees the same totals live.

We use a free **Google Sheet + Apps Script** as the tiny backend.

## 1. Create the sheet
1. Go to <https://sheets.new> (creates a new Google Sheet).
2. Name it anything, e.g. **Cesspool Drinks**.

## 2. Add the script
1. In that sheet: **Extensions → Apps Script**.
2. Delete whatever code is there and paste **all** of this:

```javascript
// Cesspool drinks counter backend (beer / shot / wine)
var TYPES = ['beer', 'shot', 'wine'];

function getSheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sh = ss.getSheetByName('Drinks');
  if (!sh) {
    sh = ss.insertSheet('Drinks');
    sh.appendRow(['name', 'beer', 'shot', 'wine']);
    // carry over counts if you previously used the beer-only 'Beers' sheet
    const old = ss.getSheetByName('Beers');
    if (old) {
      const r = old.getDataRange().getValues();
      for (let i = 1; i < r.length; i++) if (r[i][0]) sh.appendRow([r[i][0], Number(r[i][1]) || 0, 0, 0]);
    }
  }
  return sh;
}
function readAll() {
  const rows = getSheet().getDataRange().getValues();
  const out = {};
  for (let i = 1; i < rows.length; i++) {
    if (!rows[i][0]) continue;
    out[rows[i][0]] = { beer: Number(rows[i][1]) || 0, shot: Number(rows[i][2]) || 0, wine: Number(rows[i][3]) || 0 };
  }
  return out;
}
function json(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
function doGet() { return json({ ok: true, drinks: readAll() }); }
function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.waitLock(5000);
  try {
    const body = JSON.parse((e && e.postData && e.postData.contents) || '{}');
    const type = TYPES.indexOf(body.type) !== -1 ? body.type : 'beer';
    const col = TYPES.indexOf(type) + 2; // beer=2, shot=3, wine=4
    if (body.action === 'add' && body.name) {
      const sh = getSheet();
      const rows = sh.getDataRange().getValues();
      const delta = Number(body.delta) || 0;
      let found = false;
      for (let i = 1; i < rows.length; i++) {
        if (rows[i][0] === body.name) {
          sh.getRange(i + 1, col).setValue(Math.max(0, (Number(rows[i][col - 1]) || 0) + delta));
          found = true; break;
        }
      }
      if (!found) {
        const row = [body.name, 0, 0, 0];
        row[col - 1] = Math.max(0, delta);
        sh.appendRow(row);
      }
    }
    return json({ ok: true, drinks: readAll() });
  } finally {
    lock.releaseLock();
  }
}
```

3. Click **Save** (💾).

## 3. Deploy it as a web app
1. Click **Deploy → New deployment**.
2. Click the gear ⚙ next to "Select type" → choose **Web app**.
3. Set:
   - **Execute as:** *Me*
   - **Who has access:** **Anyone**   ← important
4. Click **Deploy**. Approve the permissions prompt (it's your own script).
5. Copy the **Web app URL** it gives you. It looks like:
   `https://script.google.com/macros/s/AKfy....../exec`

## 4. Paste the URL into the site
1. Open `index.html` in the repo.
2. Find this line near the drinks code:
   ```javascript
   const BEER_API = "";
   ```
3. Put your URL between the quotes:
   ```javascript
   const BEER_API = "https://script.google.com/macros/s/AKfy....../exec";
   ```
4. Commit/push to `main` (or just ask me — paste me the URL and I'll wire it in).

That's it. The Drinks tab will switch from "🟠 this phone only" to
"🟢 Synced for everyone — live", and every tap (🍺 / 🥃 / 🍷) updates the
shared Google Sheet for all your friends.

### Notes
- Anyone with the dashboard link can add a drink — fine for a friends' pool,
  no logins. The sheet only stores names + counts.
- If you ever change the script, do **Deploy → Manage deployments → Edit → New version**
  so the same URL keeps working.
- Already set up the older **beer-only** sheet? Just replace the script with the
  one above and redeploy a **New version** — it adds the shot/wine columns
  automatically and keeps your existing beer counts.
