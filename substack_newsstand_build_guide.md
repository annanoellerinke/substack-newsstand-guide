# Substack Newsstand Build Guide

### Make your writing feel like a collection someone can pick up and browse.

Based on [Le Index](https://le-index.vercel.app), built for [Anna Noelle Rinke](https://annanoellerinke.substack.com/).

Give this document to an AI coding tool or a developer, along with your publication details. It is a complete implementation brief, not a code package. The builder should create the application, import your real writing, show you the result, and help connect and publish it.

You supply your identity, content access, and preferences. The builder handles the implementation. Keep your own publication's character: the example colors and copy below are starting points, not a requirement to imitate Le Index's identity.

## 1. Start here: your publication brief

Copy this, fill in what you know, and attach your logo and reference images:

```text
Publication name:
Author name, exactly as it should appear:
Substack URL:
One-sentence description:
Logo/wordmark: [attached, or public URL]
Text and background colors: [match my Substack, or specify]
Visual references: [attach images or links]
Reader suggestions: [yes, connect to my Notion / no]
Notion parent page: [link, if using suggestions]
Preferred website address: [existing domain / decide later]
Anything that must stay exactly as written:
```

If you do not have a logo, ask for a typeset wordmark in your own publication name. If your images are low resolution, use them at a smaller size or supply a better original.

### The prompt to send with this guide

```text
Build my publication a magazine newsstand website using the attached
Substack Newsstand Build Guide and my publication brief.

Use my real Substack articles, tags, dates, links, and each article's own
share images. I want a considered editorial design: a simple offwhite
background, a centered publication logo, worn wooden shelves, and magazines
with different proportions, heights, subtle paper edges, and natural lean.
Aim for seven to eight magazines across on a large desktop, with fewer on
smaller screens. Never show more than two consecutive covers in the same
layout style, including after filtering or changing the sort.

Add topic filters and Newest first / Most popular sorting. Popularity must
come from a real source, not an invented score. If I enable suggestions,
place a small form near the top and connect it to my own private Notion
database through a server endpoint.

Read the guide, inspect my source content, ask only for missing decisions,
and show me a plan before building. Then implement, test on desktop and
phone, and show me the preview. Tell me clearly what is live, what is a
fallback, and what still needs my account access. Never claim a form is
connected until a real submission has arrived in the destination.
```

## 2. What you are making

A browsable archive where each article appears as a physical magazine on a wooden rack. Visitors can filter by topic, sort the collection, open a magazine preview, and continue reading on Substack. A small invitation near the top lets them suggest what you should write about next.

The core is the archive and newsstand. The Notion suggestion form is optional. The website does not need an AI model at runtime, an AI API key, a new subscriber system, or a second copy of your paid articles. Substack remains the place readers subscribe and read the full work.

### Reference implementation

Le Index uses Next.js App Router, React, TypeScript, custom CSS, and Vercel hosting. Its reader suggestions go straight from a server route to Notion. Other frameworks are fine if they can provide the same rendering, caching, and server-side form handling. A purely static site needs an additional server function for Notion submissions.

Use supported framework versions and a lockfile. Do not change an existing project's stack simply to match these filenames.

## 3. The visual direction

Think of a small, carefully arranged magazine shop: tactile paper, aged oak, an uncluttered wall, and covers with enough individuality to invite browsing.

The realism comes from small, consistent details. Magazines sit on a shared shelf baseline, but their tops land at different heights. A thin page block and a slight fold shadow give each cover depth. The shelf casts a soft shadow onto the background. Wood has uneven grain, faded edges, and small imperfections.

### Page composition

| Area | Treatment |
| --- | --- |
| Top navigation | Author name on the left, “A Substack Publication” in the center, Subscribe on the right. Keep all three restrained and consistent. |
| Publication identity | Centered logo/wordmark, with the author's name below when it is part of the identity. No extra “Welcome to” heading. |
| Introduction | Small reader invitation on the left; publication description on the right. Align the top of the description with the invitation heading. |
| Suggestion input | Under its heading, with a thin underline and small send control. It should not dominate the page. |
| Archive controls | Real topic filters and a quiet sort dropdown immediately above the shelves. |
| Collection | Multiple wooden shelves, naturally varied magazines, comfortable space between rows. |
| Article preview | Larger cover, title, date, subtitle, topic, access label where relevant, and Read on Substack. |

On narrow screens, stack the introduction, wrap or scroll the controls accessibly, and reduce the magazine count. Preserve readable type and usable touch targets.

### Colors and typography

Match the publisher's actual Substack palette when requested. Sample the real page or supplied assets rather than choosing a vaguely similar theme.

Le Index's reference palette is `#f8faff` for the background and `#37425d` for primary text. It uses Cormorant Garamond for display text, Karla for interface/body text, and Space Mono sparingly for small catalogue details. The publication wordmark is its original asset, not an approximation in the display font.

Keep the logo modest enough to stay sharp. In the reference build, roughly 330 CSS pixels on desktop and 280 on mobile worked well; use the actual asset's resolution to decide. Give it descriptive alternative text.

### Details that make it feel bespoke

- Use each article's real share artwork rather than repeating a stock image.
- Preserve cover proportions instead of stretching all covers into one rectangle.
- Vary height, width, and lean slightly, consistently per article.
- Use thin paper edges, a subtle spine crease, and light catching one edge.
- Give the shelf a back rail, small side pieces, and a shallow front lip.
- Use a licensed, owned, or generated wood texture; vary its crop so each row does not look stamped out.
- Keep shadows soft and directional. Heavy plastic gloss defeats the paper effect.
- Leave the cover faces unobstructed: the final Le Index rack has no horizontal black retaining wires.
- Avoid repeating the author's name on every magazine preview.
- Use a restrained hover lift and a gentle preview transition; support reduced motion.

## 4. Import the complete Substack archive

The publication is the source of truth. Import actual titles, subtitles, publication dates, canonical article links, visible tags, cover images, post IDs, and free/paid status. Do not invent missing articles or copy paywalled bodies into the site.

A normalized model can look like this:

```ts
export type Article = {
  id: string;              // stable slug or publication-specific ID
  postId?: string;         // used for share assets when available
  title: string;
  subtitle: string | null;
  date: string;            // ISO date
  url: string;             // canonical article URL
  isPaid: boolean;
  tags: string[];
  coverImage: string | null;
};
```

### Data access and pagination

The reference build retrieved public archive metadata from this observed Substack endpoint:

```text
https://YOUR-PUBLICATION.substack.com/api/v1/archive?sort=new&limit=50&offset=0
```

This is an implementation detail observed during the build, not a guaranteed public API contract. Verify its current behavior for the target publication. If unavailable, use an owner-provided export or another verified source. An RSS feed may be useful for updates but should not be assumed to contain the entire historical archive.

The import must:

1. Fetch a batch and validate its shape.
2. Normalize and deduplicate entries by stable ID.
3. Advance the offset by the number actually received.
4. Continue until an empty batch, not merely a batch shorter than the requested limit.
5. Stop safely if pagination repeats without adding records.
6. Save a complete snapshot only after the whole import succeeds.

In our build, Substack returned fewer posts than the requested limit before the archive was complete. Treating a short page as the end silently dropped articles.

Validate article destinations against the publication's known domains. If it uses a custom domain, explicitly support both verified domains rather than rejecting valid canonical links or accepting arbitrary hosts.

### Tags and “General”

Generate filters from real, visible tags. One article may belong to more than one filter; those counts need not sum to the total number of articles.

If something appears in General even though it is tagged in Substack, compare the dashboard with the actual returned metadata and refresh the cache. A screenshot or owner confirmation can justify a small fallback map keyed by article ID. Use it only when the source tags are empty, and let future nonempty source tags take precedence. Never guess categories merely to remove General.

### Updating and fallbacks

The reference website revalidates archive data on requests after an hourly cache interval and retains a complete saved snapshot for failures. This is request-triggered caching, not a background job running every hour.

Store snapshot timestamps and make stale-data conditions visible in maintenance diagnostics. When neither a live archive nor a saved snapshot exists, show an honest empty/error state. Do not manufacture a full collection.

## 5. Make each article a magazine

### Get the right images

Use the article's own Substack promotional/share images, including the text treatment. The author's Share assets screen offers variations; download the individual images rather than using a screenshot of the dashboard.

The reference build observed assets in this form:

```text
https://YOUR-PUBLICATION.substack.com/api/v1/press_kit/asset/POST_ID/STYLE/composed?aspectRatio=RATIO
```

Observed styles were `overlay`, `split`, and `title`; ratios included `grid` (4:5) and `stories` (9:16). These routes are also undocumented implementation details. Confirm availability, dimensions, and article identity before using them. If they stop working, use the owner's downloaded share assets or create approved cover treatments using the article's actual image and title.

Cache approved covers locally, optimize them, and keep a manifest recording article ID, source URL, style, dimensions, and local path. A 720-pixel-wide WebP was sufficient for many reference assets; check the enlarged preview before settling on compression. Respect rate limits, use low concurrency, and skip already cached files when resuming.

### Preserve variety without randomizing the page

Derive physical variation from a stable hash of the article ID. A reload should not give the same issue a completely different shape.

Useful starting values from the reference implementation:

| Property | Starting range |
| --- | --- |
| Tall cover layout weight | 108–132 |
| Wider cover layout weight | 138–175 |
| Lean | −1.5° to +1.5° |
| Visible paper thickness | 2–4 CSS pixels |
| Cover proportions | Preserve 9:16 and 4:5 variants |

Widths here are relative layout weights; the row scales to fit. Preserve proportions when scaling. Choose the actual asset first, then use its dimensions: do not force a wide image into a tall ratio because a different variant was expected.

### No more than two consecutive matching styles

The final rule is about consecutive cover layouts, not a limit of two uses across the entire collection. Apply it to the visible ordered articles after both filtering and sorting:

```text
Walk through the visible articles in order.
Choose each article's preferred available cover style.
If that would make three of the same style in a row,
choose a different available variant for that same article.
Record the chosen variant and continue.
```

Render the matching image, not just a different style label. If variants are unavailable, acquire them or use an approved fallback and report the limitation. Do not silently claim the variety rule is satisfied.

### Fit seven or eight across

Measure the rack's available width rather than relying only on the browser width. On a large desktop, aim for seven or eight magazines per full shelf, using fewer when wider issues need room. Balance the last rows without inventing duplicates or filler magazines.

For example, a collection of 63 articles can be arranged as seven rows of eight and one of seven. A filtered collection with only four results should simply show four.

The reference used capacities of eight at rack widths of at least 950px, seven at 650px, four at 460px, three at 360px, and two below that. These are starting points; verify actual rendered covers and page padding on phones.

## 6. Browsing, filtering, and popularity

Default to **Newest first**. Offer **Most popular** in a small dropdown.

The reference retrieved a separate ordered archive with `sort=top`. Preserve that returned order and map its IDs onto the same article collection. Do not sort the result back into date order. Validate this source against the publication's own popular listing before calling it Most popular.

Substack's ranking is not a claim about exact views, subscriber conversions, or revenue. Do not invent those metrics. If a dependable popularity source is unavailable, omit the option until the owner supplies one, or offer an explicitly named editorial selection such as Editor's picks.

Keep a saved ranking fallback. If newly published articles are missing from the ranking, append them predictably, for example newest first after ranked entries. Topic filters should work in both sorting modes and should not change the selected sort.

### Opening an article

Make the entire magazine a keyboard-accessible button. Keep its hit area stable while an inner visual layer moves on hover. Clicking or tapping opens a preview with the correct cover, exact title, date, subtitle, and link to the original post.

Use a native dialog or equivalent accessible modal. Focus moves into it, Escape closes it, Tab stays within it, and closing restores focus to the originating magazine. Mobile readers must be able to open and dismiss it without hover. Mark decorative textures as hidden from assistive technology.

## 7. Reader suggestions at the top

Place this above the collection, not at the bottom. Keep it compact so the magazines remain the focus.

Reference copy, which you may customize:

| Element | Copy |
| --- | --- |
| Heading | What should I write about next? |
| Input placeholder | Something you’re figuring out, something you’re curious about, or something you want my take on. |
| Send button | Send it my way ↗ |
| Confirmed success | Got it. Thank you for giving me something to think about. |

Put the input directly beneath the heading, with a fine underline and left-aligned text. Do not repeat the placeholder as another paragraph. Give the field a persistent accessible label because placeholder text disappears during typing.

Show sending, success, and error states. Keep the message if submission fails. Clear it only after the server confirms delivery. If the backend is not configured, disable the form and explain that suggestions are not open yet.

## 8. Connect your own Notion database

Create a new private database inside the owner's chosen page. Do not reuse the example publication's database or credentials.

### Schema

| Property | Type | Purpose |
| --- | --- | --- |
| Suggestion | Title | Short label derived from the message |
| Message | Rich text | Full reader message |
| Submitted | Created time | Submission timestamp |
| Status | Select | New, Considering, Written |
| Submission ID | Rich text | Stable retry identifier |
| Client Hash | Rich text | Optional pseudonymous abuse-control identifier |

Create a dedicated connection with read and insert access to this database only. Read access is needed for the reference retry and rate checks. Update, comment, and user-profile access are not needed for this design. Verify the connection's actual content access in Notion; possessing a token alone is not enough. Notion's portal and credential options change, so consult its [authentication documentation](https://developers.notion.com/reference/authentication) when setting it up.

Store the credential in the hosting provider's server environment, never in browser JavaScript or a public repository. For this implementation the names are:

```dotenv
NOTION_API_TOKEN=replace_in_your_host_secret_settings
NOTION_DATA_SOURCE_ID=your_own_data_source_id
SITE_URL=https://your-site.example
LAUNCH_READY=false
```

These names are application conventions. `LAUNCH_READY` only controls indexing if the builder implements that behavior. Keep actual local values in ignored `.env.local`; commit only a placeholder `.env.example`.

A Notion database ID and its data source ID are different concepts in the newer API. Resolve the intended data source, inspect its schema, and pin an API version compatible with your implementation. The reference used `2025-09-03`; do not blindly replace it with another version without checking the request shapes. See Notion's [database/data-source migration documentation](https://developers.notion.com/docs/upgrade-guide-2025-09-03).

### Submission flow

```text
Reader enters a suggestion
  → browser POSTs to the website's own server route
  → server validates and checks retries / abuse limits
  → server creates a page in the private Notion data source
  → Notion confirms the write
  → browser displays success
```

A `POST /api/recommend` route is sufficient. It should accept a stable UUID, the message, and a hidden honeypot field; reject invalid input; enforce a message limit (2,000 characters in the reference); and bound request sizes and timeouts.

Validate the request origin against the deployed site's trusted host configuration. On serverless hosts, the runtime's internal URL may differ from the public host. Test the actual deployment rather than assuming localhost behavior is representative.

Before writing, query for the Submission ID so sequential retries return success without creating another entry. For lightweight abuse control, the reference checks recent submissions from a keyed hash of the client address, with a cap of five per hour. Only use an address header known to be set by the trusted host; do not trust arbitrary forwarded headers. Avoid logging message bodies or raw addresses.

Notion's query-then-create flow is not atomic: simultaneous requests can race. Treat this as a soft limit and sequential retry protection. For stronger guarantees, use transactional idempotency storage and a hosting-level rate limiter. An anonymous endpoint cannot be secured by origin checks or a honeypot alone.

Do not expose a public endpoint that lists suggestions. If collecting additional personal information, explain its purpose and storage to readers.

## 9. Suggested implementation map

These are files the builder should create, not files bundled with this guide:

```text
app/
  page.tsx                 Load archive, rankings, and form availability
  layout.tsx               Fonts, metadata, canonical origin, indexing
  globals.css              Newsstand, magazine layers, responsive styles
  opengraph-image.tsx       Social sharing image
  api/recommend/route.ts    Server-only suggestion endpoint
components/
  Newsstand.tsx            Filters, sort, rows, article dialog
  SuggestionForm.tsx        Input and submission states
lib/
  archive.ts               Normalize and paginate source content
  substack.ts              Live fetch, caching, saved fallbacks
  magazines.ts             Stable dimensions and cover-style assignment
  recommend.ts             Validation and Notion delivery
  publication.ts           Owner-editable identity and source configuration
data/
  archive-snapshot.json
  popular-snapshot.json
  cover-manifest.json
public/assets/
  wordmark.*
  worn-wood.*
  covers/
scripts/
  sync-snapshot.*
  sync-popular.*
  sync-covers.*
.env.example
```

Centralize the publication URL, author name, description, links, palette, and asset paths so a new owner does not need to hunt through components. Do not hardcode Le Index's domains or identity into a reusable implementation.

### Build sequence

1. Inspect the publication and references; resolve missing essentials and present the plan.
2. Import and validate the complete archive and save a fallback.
3. Obtain cover variants and create the asset manifest.
4. Build one convincing shelf before laying out the entire collection.
5. Add the final header, introduction, responsive rows, filters, and real popularity sorting.
6. Add article previews and keyboard/touch behavior.
7. Connect the optional Notion form using the owner's accounts.
8. Verify the local production build and a hosted preview.
9. Publish and test the live form, article links, and sharing preview.

## 10. Publish and verify

Use a host that supports the framework and its server routes. Le Index uses Vercel. Add the server environment variables to the correct deployment environment and redeploy after changes so the running application receives them. See [Vercel's environment-variable documentation](https://vercel.com/docs/environment-variables).

Set the final site URL for canonical metadata and absolute sharing-image links. Create a clear page title, description, favicon, and social preview using the owner's identity. Allow indexing when the owner is ready to launch; a preview's noindex setting should not accidentally remain on the public site.

### Acceptance checklist

- [ ] Article count matches the verified complete source.
- [ ] Every title, destination, date, and access label is real.
- [ ] Tags and filter counts are checked, including untagged posts.
- [ ] Popularity order follows a verified source and survives filtering.
- [ ] Full desktop rows hold seven or eight issues where space permits.
- [ ] Magazine shapes vary without stretching or clipping artwork.
- [ ] No three consecutive visible covers use the same layout when variants are available.
- [ ] Covers belong to their corresponding articles.
- [ ] No horizontal black wires cross the rack.
- [ ] Wordmark is sharp and proportionate.
- [ ] At 320px, 375px, and 430px viewport widths, the page has no accidental horizontal overflow.
- [ ] Filters, select controls, and form remain readable and touchable on phone.
- [ ] Dialog works with keyboard, touch, Escape, and focus return.
- [ ] Reduced-motion preference is respected.
- [ ] Missing artwork and unavailable data have honest fallbacks.
- [ ] A uniquely labeled live-form test arrives in the intended Notion database with exact text.
- [ ] Sequential retry with the same ID leaves one entry.
- [ ] Empty input, missing configuration, timeout, and service failure do not show false success.
- [ ] Secrets and private reader data are absent from client code, logs, and committed files.
- [ ] Production build and meaningful data/form tests pass.
- [ ] Social preview, subscription link, canonical URL, and indexing settings match the published site.

Record what was actually checked. Do not claim a Lighthouse score or device test that was not performed. Label test submissions clearly so the owner can remove them later.

## 11. Keeping it up to date

Metadata refresh and cover caching are separate processes. New articles may appear through live archive revalidation while their images still use remote source URLs. If you want every image cached locally, run a scheduled asset sync or refresh and redeploy periodically.

Give the owner a short maintenance note explaining how to refresh the archive snapshot, popularity snapshot, and cover variants; where suggestions arrive; how to rotate the Notion credential; and what to do if a Substack endpoint changes. The next maintainer should not need the original chat history.

For a public repository, include this guide, a README, and placeholder configuration. Only include actual site source or assets after checking that they are intended for sharing. Exclude `.env*` secrets, host credentials, private exports, reader submissions, build output, and machine-specific files. Reuse the design approach with your own content and assets.

## Credits and scope

This guide documents the newsstand built for Anna Noelle Rinke's Le Index. The reusable build-brief format was inspired by [Carol's Virtual Library Build Guide](https://github.com/carollia99/virtual-library-guide/blob/main/virtual_library_build_guide.md); its bookshelf implementation is a separate project.

Implementation observations reflect the September 2026 Le Index build. Third-party endpoints and account interfaces can change. Verify them when building your own site.
