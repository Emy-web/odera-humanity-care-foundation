# ODERA HUMANITY CARE FOUNDATION — Website

A 10-page static website for **ODERA HUMANITY CARE FOUNDATION**, built with plain HTML, CSS and JavaScript, and hosted on AWS (S3 + CloudFront).

---

## Table of Contents

- [Overview](#overview)
- [Pages](#pages)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Editing Content](#editing-content)
  - [Editing Team, Gallery, and Events](#editing-team-gallery-and-events)
  - [Editing Other Pages (About, Projects, etc.)](#editing-other-pages-about-projects-etc)
- [Deploying to AWS](#deploying-to-aws)
  - [1. Create an S3 Bucket](#1-create-an-s3-bucket)
  - [2. Create a CloudFront Distribution](#2-create-a-cloudfront-distribution)
  - [3. Register a Domain with Route 53](#3-register-a-domain-with-route-53)
  - [4. Request an SSL Certificate (ACM)](#4-request-an-ssl-certificate-acm)
  - [5. Connect the Domain to CloudFront](#5-connect-the-domain-to-cloudfront)
- [Regular Maintenance (Adding Photos & Videos)](#regular-maintenance-adding-photos--videos)
- [Common Issues](#common-issues)
- [License](#license)

---

## Overview

This site was built to give ODERA HUMANITY CARE FOUNDATION an online presence covering its mission, team, programs, projects, and yearly annual events — including support for large yearly video uploads without needing a backend server or database. All content is either plain HTML or driven by one editable JavaScript file (`js/content-data.js`), so the site can be updated without a rebuild or a developer.

## Architectural Diagram Of The Website


## Pages

| Page | File | Purpose |
|---|---|---|
| Home | `index.html` | Landing page, mission summary, stats, CTA |
| About | `about.html` | Founding story, mission & vision, values |
| Our Team | `team.html` | 12 team member profiles |
| Programs | `programs.html` | Core humanitarian program areas |
| Projects | `projects.html` | Completed/ongoing project write-ups |
| Gallery | `gallery.html` | Photo gallery from outreach and events |
| Events | `events.html` | Annual events with embedded video recordings |
| Get Involved | `get-involved.html` | Volunteer/partner sign-up |
| Donate | `donate.html` | Donation tiers and form |
| Contact | `contact.html` | Contact form and details |

## Project Structure

```
odera-site/
├── index.html
├── about.html
├── team.html
├── programs.html
├── projects.html
├── gallery.html
├── events.html
├── get-involved.html
├── donate.html
├── contact.html
├── css/
│   └── style.css
├── js/
│   ├── content-data.js     ← editable content (team, gallery, events, programs)
│   └── script.js            ← renders content-data.js into the pages
├── images/
│   ├── team/
│   ├── gallery/
│   ├── events/
│   └── (custom folders as needed, e.g. about/, projects/)
└── README.md
└── .gitignore

```

## Tech Stack

- HTML5, CSS3 (no framework, no build step)
- Vanilla JavaScript (`js/script.js`, `js/content-data.js`)
- Google Fonts (Fraunces, Work Sans)
- Hosted on **AWS S3** (storage) + **AWS CloudFront** (CDN/HTTPS)
- Domain via **AWS Route 53**
- SSL via **AWS Certificate Manager (ACM)**

## Editing Content

### Editing Team, Gallery, and Events

These three sections are **data-driven** — you only edit lists inside `js/content-data.js`, and the page renders them automatically. No HTML editing required.

1. Upload the photo/video file into the matching folder (`images/team/`, `images/gallery/`, or S3's `videos/` folder for event videos).
2. Open `js/content-data.js` and find the relevant list: `TEAM_MEMBERS`, `GALLERY_ITEMS`, or `EVENTS`.
3. Fill in the fields for that entry, e.g.:
   ```js
   { name: "Jane Doe", role: "Coordinator", photo: "images/team/jane-doe.jpg", bio: "..." }
   ```
4. The `photo` (or `videoUrl`) path must always start with `images/...` (for local files) or a full `https://` CloudFront URL (for videos) — **never** a path from your own computer like `C:\Users\...`.
5. Save the file (`Ctrl+S`).

### Editing Other Pages (About, Projects, etc.)

Pages like `about.html` and `projects.html` are hand-written HTML, so photos are added directly as an `<img>` tag rather than through `content-data.js`:

```html
<div class="team-photo" style="aspect-ratio:4/3;">
  <img src="images/about/founding-photo.jpg" alt="Founding photo">
</div>
```

Key rules:
- Typing a file path as plain text does **not** display an image — it must be wrapped in an `<img src="...">` tag.
- The folder named in `src="images/..."` must match the folder the file is actually sitting in.
- Avoid spaces and double extensions in filenames (e.g. `photo.jpg.jpg`) — use simple names like `event-2026.jpg`.

## Deploying to AWS

### 1. Create an S3 Bucket

1. Go to **S3 → Create bucket**.
2. Give it a unique name (e.g. `odera-humanity-foundation-site`).
3. Block all public access (CloudFront will access it privately via OAC — see below).
4. Upload the full contents of this repository to the bucket root, keeping the folder structure (`css/`, `js/`, `images/`, and the `.html` files).

### 2. Create a CloudFront Distribution

1. Go to **CloudFront → Create distribution**.
2. Set the **Origin domain** to your S3 bucket (use the **S3 REST endpoint**, not the website endpoint).
3. Under **Origin access**, choose **Origin Access Control (OAC)** and create a new OAC.
4. Copy the bucket policy CloudFront generates and paste it into your S3 bucket's **Permissions → Bucket Policy**.
5. Set **Default root object** to `index.html` — this is required or you'll get an `AccessDenied`/blank page at the domain root.
6. Wait for the distribution status to show **Enabled/Deployed**, then test the auto-generated `*.cloudfront.net` domain.

### 3. Register a Domain with Route 53

1. Go to **Route 53 → Domains → Register domain**.
2. Search for and register your desired domain (e.g. `oderahumanitycare.org`).
3. Once registered, Route 53 automatically creates a **Hosted Zone** for the domain — this is where DNS records (from ACM validation and CloudFront) will be added later.

### 4. Request an SSL Certificate (ACM)

1. Go to **AWS Certificate Manager**, and switch region to **US East (N. Virginia) — us-east-1** (required for CloudFront regardless of where your bucket/domain is).
2. **Request a public certificate** for your domain and its `www` subdomain (e.g. `oderahumanitycare.org` and `www.oderahumanitycare.org`).
3. Choose **DNS validation**.
4. Since the domain is on Route 53, click **Create records in Route 53** directly from the ACM screen — it adds the validation CNAME records automatically.
5. Wait for the certificate status to change to **Issued**.

### 5. Connect the Domain to CloudFront

1. In your CloudFront distribution → **General → Edit**.
2. Add your domain and `www` subdomain under **Alternate domain name (CNAME)**.
3. Select the ACM certificate you just issued under **Custom SSL certificate**.
4. Save and wait for the distribution to redeploy.
5. In **Route 53 → Hosted Zone**, add an **A record (Alias)** for both the root domain and `www`, pointing to your CloudFront distribution.
6. Test `https://yourdomain.org` once DNS propagates.

## Regular Maintenance (Adding Photos & Videos)

This site is designed for **yearly updates** after each annual event:

1. **Photos** — upload new images to the relevant `images/` subfolder (locally in the project), add/edit an entry in `js/content-data.js` (for Team/Gallery/Events) or directly as an `<img>` tag (for other pages).
2. **Videos** — upload the video file directly to the `videos/` folder in your **S3 bucket** (not this local project — video files are too large to keep in the repo). Copy the resulting CloudFront URL, e.g.:
   ```
   https://your-distribution-id.cloudfront.net/videos/2027-annual-event.mp4
   ```
   Paste it into that event's `videoUrl` field in `js/content-data.js`.
3. **Publish the update**:
   - Upload the changed file(s) to your S3 bucket, keeping the same folder structure.
   - Go to **CloudFront → Invalidations → Create Invalidation**, and enter `/*` (or the specific changed file path) to clear the cache so visitors see the update immediately.
4. **Adding a brand-new page** (e.g. a dedicated page for a specific year's event): duplicate the closest existing `.html` file, edit its content, link to it from a relevant page, and optionally add it to the navigation menu across all pages.

## Common Issues

| Symptom | Likely Cause |
|---|---|
| `AccessDenied` XML error on the CloudFront domain | Default root object not set to `index.html`, or bucket policy/OAC not linked correctly |
| Team/Gallery/Events section is completely empty | A syntax error in `js/content-data.js` (often a missing quotation mark) — check the browser console for the exact line |
| A person's name shows instead of their photo | The `photo:` path doesn't match the actual file location/name, or it points to a local PC path (`C:\...`) instead of `images/...` |
| Image shows as a small broken icon | Same as above — path/filename mismatch, wrong folder, or a double file extension (`.jpg.jpg`) |
| Changes not visible after upload | CloudFront is still serving a cached version — create an invalidation for `/*` |

## License

© ODERA HUMANITY CARE FOUNDATION. All rights reserved.

This repository and its content (text, images, and video assets) are the property of ODERA HUMANITY CARE FOUNDATION and may not be reused, redistributed, or repurposed without written permission.

The underlying code structure (HTML/CSS/JS templates, excluding foundation-specific content and media) may be reused under the [MIT License](https://opensource.org/licenses/MIT) if you'd prefer an open license instead — replace this section with the standard MIT text if so.
