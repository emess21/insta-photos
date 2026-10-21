# Automated Instagram & Facebook Posting From a Photo Folder

Post one photo from a large folder to Instagram and a Facebook Page, twice a day, with no manual work after setup. Photos are prepared by a small desktop tool, hosted on GitHub, queued in a Google Sheet, and published on a schedule by Make.com. Every part runs on a free tier.

```
 Original photos
       │
       ▼
 Photo Studio (desktop app)        sorts by Date Taken, renames, resizes, borders, pads to square
       │
       ├──► GitHub repo (public)   hosts the processed images at stable URLs
       │
       └──► Queue CSV ──► Google Sheet   one row per post: filename, photo URL, status, caption
                              │
                              ▼
                     Make.com scenario (runs on a schedule)
                       1. read the next "pending" row
                       2. post to Instagram
                       3. mark the row "posted"
                       4. post to the Facebook Page
```

## What to expect from this

This setup is good for **steady posting, brand visibility and driving visitors to your sites**. It is **not** an SEO backlink tactic. Links and hashtags on Instagram and Facebook are `nofollow` or not links at all, and search engines give them little or no ranking weight. Treat it as an audience-building tool.

## Requirements

| What | Notes |
|---|---|
| Instagram **Business** account | Creator accounts are **not** supported by Make's Instagram module |
| Facebook **Page** | A Page, not a personal profile. Your Instagram account must be linked to it, and you must be an admin |
| GitHub account | Free. Hosts the images in a **public** repository |
| Google account | For the Google Sheet |
| Make.com account | Free plan is enough (see limits below) |
| Python 3 with Pillow | To run Photo Studio from source, or build it into an executable |

Free-tier figures at the time of writing (check the current pricing pages): Make's free plan gives roughly 1,000 credits a month, 2 active scenarios and a 15-minute minimum run interval. This workflow uses about 4 credits per run, so two runs a day is around 250 a month.

## Files

| File | Purpose |
|---|---|
| `photo_studio.py` | Desktop app (tkinter + Pillow). Tab 1 prepares photos, tab 2 edits the post queue |
| `square_pad.py` | Standalone version of the pad-to-square step (also built into Photo Studio) |
| `build_linux.sh`, `build_windows.bat`, `build_mac.sh` | Build a single-file executable for each platform |
| `build.yml` | Optional GitHub Actions workflow that builds all three for you |

## One-time setup

### 1. Accounts

1. Switch Instagram to a **Business** account: Settings and privacy → Account type and tools → Switch to professional account → Business.
2. Create a Facebook Page (facebook.com/pages/create) if you don't have one.
3. Link them: on the Page, Settings → Linked accounts → Instagram → Connect. Check both appear connected in Meta Business Suite.

### 2. GitHub image host

1. Create a **public** repository, for example `your-photo-repo`.
2. Images will be served from `https://raw.githubusercontent.com/YOUR-GITHUB-USERNAME/your-photo-repo/main/photo001.jpg`. This URL pattern is predictable from the filename and returns the raw image, which Instagram's publishing API needs.

### 3. Prepare photos with Photo Studio

Run it with `python photo_studio.py` (after `pip install Pillow`), or build an executable with the script for your platform.

On the **Prepare Photos** tab:

| Setting | Recommended |
|---|---|
| Input folder | Folder with your original photos |
| Output folder | A new empty folder for processed photos |
| Sort new photos by | `date taken` (EXIF Date Taken, oldest first) |
| Max edge / Border | 2000 px / 40 px |
| Pad to square | On (black background, nothing cropped) |
| Allow upscaling | Off |

Click **Run**. Photos are numbered `photo001.jpg`, `photo002.jpg` and so on in Date Taken order. Photos with no Date Taken metadata fall back to their file-modified date, and the log tells you which ones.

Photo Studio keeps a small manifest file in the output folder recording what has been processed, so re-running only handles new photos (see *Adding new photos* below).

Upload the processed photos to the GitHub repo (Add file → Upload files; the browser uploader handles about 100 files at a time).

### 4. Build the queue

On the **Content Queue** tab:

1. Enter your GitHub username and repo name and click **Save settings**.
2. Back on tab 1, click **Send new photos to Content Queue →**. Each photo gets a row with its URL and a caption.
3. Edit captions as you like (click a row, edit, save).
4. Click **Regenerate all photo URLs**, then **Save queue CSV**.

Captions follow a fixed layout, with a block of invisible spacer lines so everything after the hashtags sits behind Instagram's "… more":

```
#tag1 #tag2 #tag3 …
⠀
⠀   (invisible lines)
⠀
Your copy here
```

The hashtags and the list of rotating copy are defined near the top of `photo_studio.py` (the `FIXED_TAGS` line and the list of copy variations below it). Replace them with your own.

### 5. Google Sheet

1. Import the queue CSV into a new Google Sheet (File → Import → Upload).
2. Columns should be: `post_number`, `filename`, `photo_url`, `status`, `caption`.
3. Type `pending` in the first status cell, select the whole status column down to the last row, and press **Ctrl+D** to fill down.

Make filters on the exact text `pending`, so every unposted row needs it.

### 6. Make.com scenario

Create a new scenario with four modules in this order:

1. **Google Sheets → Search Rows.** Select the spreadsheet and sheet. Filter: `status` equal to `pending`. Limit: 1. Sort by `post_number` ascending.
2. **Instagram for Business (Facebook login) → Create a photo post.** Connect with Facebook and, in the permissions dialog, tick your Page and your Instagram account. Choose your Page in the dropdown. Map **Photo URL** to column C (`photo_url`) and **Caption** to column E (`caption`).
3. **Google Sheets → Update a Row.** Same spreadsheet and sheet. Row number: the *Row number* output from Search Rows. Set `status` to `posted`.
4. **Facebook Pages → create a photo post.** Connect the same Facebook account, choose the Page, map the photo URL and the message (caption).

Update a Row sits **before** the Facebook step on purpose: if Facebook fails, the row is already marked posted, so the next run won't repost the same photo to Instagram.

Click **Run once** to test (it makes a real post). Then set the schedule: clock icon on the first module → *Every day* → add your two times, and switch the scenario **ON**.

## Adding new photos (quick guide)

For when you've shot more and want them added to the queue.

**Checklist**

1. Copy the new photos into your original input folder. Give them **unique filenames** (see the note below).
2. Open Photo Studio → Prepare Photos. Use the **same output folder** as before. Click **Run**. Only the new photos are processed and numbered after the existing ones.
3. Go to the **Content Queue** tab and click **Load queue CSV…** to open your existing queue file. Do this *before* the next step, otherwise you start from an empty queue.
4. Back on tab 1, click **Send new photos to Content Queue →**. Check the new rows, then **Save queue CSV**.
5. Upload only the **new** processed photos from the output folder to the GitHub repo.
6. In the Google Sheet, copy the **new rows only** from the CSV and paste them at the **bottom** of the sheet. Set their `status` to `pending`.
7. Done. Make picks them up when it reaches them.

**Important notes**

- **Never re-import the whole CSV over the sheet.** Make records `posted` in the Google Sheet, not in your local CSV, so replacing the sheet resets your posting history and photos would be reposted. Only append new rows.
- **Filenames must be unique.** Photo Studio remembers originals by filename. If a camera card is reset and new photos reuse old names such as `IMG_0001.jpg`, they're treated as already processed and skipped. Rename them first so every file has a name you haven't used before.
- **Order.** New photos are sorted by Date Taken among themselves and numbered after the existing ones, even if they were shot earlier. Renumbering would break existing GitHub links, so it is deliberate.
- **Redoing everything.** Tick *Force reprocess ALL photos* to rebuild every image from `photo001`. Re-upload them all afterwards. Only use this if the input folder hasn't changed, otherwise numbering shifts.
- **Keep a buffer.** To see how many posts are left, use `=COUNTIF(D:D,"pending")` in the sheet. At two posts a day, 60 pending rows last a month.

## Troubleshooting

| Problem | Likely fix |
|---|---|
| Page dropdown in the Instagram module is empty ("no data available") | Make wasn't granted your Page. Facebook → Settings → Business integrations → Make → remove it, then recreate the connection and tick the Page and Instagram account in the dialog. Also check you're a full admin of the Page and that Instagram is linked to it |
| `Error validating access token … (190, OAuthException)` | Facebook invalidated the login session. Open facebook.com and clear any security prompt, then in Make go to Connections → Reauthorize (or delete and recreate) |
| "The operation failed for 1 parameter(s)" | Usually the Photo URL field is empty or not a valid URL. Check column C holds a full `https://raw.githubusercontent.com/…` link and that it opens as a bare image in a browser. The repo must be public |
| Nothing posts and the run finishes quietly | No row matched the filter. Check the status cells say exactly `pending` |
| Instagram rejects some photos | Instagram's API accepts aspect ratios from about 4:5 to 1.91:1. Pad-to-square avoids this. Make sure the option is on |
| Caption spacer lines collapse | Some tools strip invisible characters. Test one post and, if needed, check the caption in Make's module output |
| `tkinter not found` on Linux | `sudo apt install python3-tk` |

## Privacy and security notes

- Do not commit tokens or passwords. Make and Meta handle authentication through their own login flows, so nothing secret lives in this project.
- The image repo is public by necessity. Only upload photos you're happy to be publicly accessible.
- Photo Studio stores your GitHub username and repo name in a settings file in your home folder (`.photo_studio_settings.json`).
