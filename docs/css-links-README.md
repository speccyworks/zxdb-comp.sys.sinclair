# comp.sys.sinclair → ZXDB links

`css-links.sqlite` (3 MB, SQLite) links posts in the Usenet newsgroup **comp.sys.sinclair** (December 1993 – August 2026) to **ZXDB**, the database behind Spectrum Computing and ZXInfo. It works like ZXDB's magazine references (`magrefs`), with the newsgroup as the source: each link says which post is a release announcement, POKE, map, solution, tip, contest entry or contest result for which ZXDB entry, and which contest posts belong to which CSSCGC year.

It also lists every thread about the TZX tape file format, checked by reading (`topic_threads`).

It uses ZXDB's own ids: `entry_id`, `label_id` and `tag_id` are ZXDB ids, and post types are ZXDB `referencetypes`.

## What is in it

| Table | Rows | Contents |
|---|---|---|
| `postrefs` | 9,914 | Post → ZXDB entry, label or tag, with the reference type, the rule that made the link (`method`), a rough confidence (`score`, 0–1), the text that matched, and extra details (the version in an announcement's subject, POKE address,value pairs, or the Tipshop contributor). 8,041 links have a score of 0.5 or more, for 3,528 ZXDB entries. |
| `tipshop_credits` | 6,432 | Who sent the Tipshop (https://www.the-tipshop.co.uk/) which POKEs, maps, solutions and tips for which game, read from the Tipshop's update posts (2003–2021): contributor as written, one name per person, the ZXDB label with that name, the game as written and its ZXDB entry. |
| `post_types` | 4,129 | What a post is, as a ZXDB reference type, and the rule that decided it. |
| `announcement_groups` | 982 | Each release announcement sorted into `sinclair` (software or hardware for Sinclair machines), `pc` (emulators and tools for other computers) or `media` (magazines, web sites, news). |
| `announce_files` | 1,934 | Versions, links and file names given in release announcements, as written; not checked. |
| `topic_threads` | 74 | Every thread about the **TZX tape file format** (first called ZXTape), found by searching and checked by reading: 43 about the format (Tomaz Kac's proposal of 1996–97, the updates and revisions 1.0–1.12, the homepage, later questions and extensions), 31 on other subjects with posts about the format. Each with its key posts, a one-line summary and the people taking part. |
| `posts` | 4,315 | Every post the tables above refer to: Message-ID, date (UTC), the poster's name, subject and thread. |
| `threads` | 1,764 | The threads those posts are in: first subject, first and last date, number of posts, Message-ID of the first post. |
| `zxdb_entries`, `zxdb_labels`, `zxdb_tags` | 4,397 / 86 / 29 | Copies of the ZXDB rows linked to (title, first release year, authors, number of magrefs; label and tag names), so the file can be read without ZXDB. ZXDB copy of 18 Sep 2026. |
| `referencetypes` | 26 | ZXDB's reference types. |
| `meta` | – | When the file was made. |

Views: `v_postrefs` (each link with the post's date, poster, subject, reference type and the ZXDB title, label or tag name) and `v_tipshop_credits` (each credit with the post's date and the ZXDB title and label).

## How the links were made

Almost all links were made by rules, not by hand, and `method` names the rule. They are candidates: none was checked one by one, except where `method` is `checked-by-hand`.

| `method` | Links | What it means |
|---|---|---|
| `tipshop-credit` | 6,360 | A game credited in a Tipshop update. |
| `csscgc-subject`, `-result`, `-entry` | 2,297 | A post about the CSSCGC (the group's Crap Games Competition), linked to that year's ZXDB contest tag. |
| `csscgc-title-in-post` | 281 | A title of that year's contest entries found in a contest post's own text. |
| `announce-title-in-subject`, `-in-text` | 391 | A ZXDB title found in a release announcement. |
| `map-`, `poke-`, `solution-title-in-subject` | 528 | A ZXDB title found in the subject of a post about a map, POKEs or a solution. |
| `checked-by-hand` | 57 | Release announcements found by reading the posts, where ZXDB lists the release under another name (e.g. "SE Basic" is *OpenSE BASIC*), and the six *Emulate!* issues. |

A title shared by several ZXDB entries lowers the score, split between them, so **use `score >= 0.5`** for links that are probably right. Lower scores are kept for completeness. Maps and solutions are typed by subject, so requests ("Anyone got a Hobbit map?") are included as well as the maps and solutions themselves.

Six announcement links that turned out to point at the wrong entry when checked are left out: *Emulate!* the magazine (linked to a 1997 demo of the same name), a Laser Squad editor (linked to another one), and *zblast 81* (linked to *Zblast SD* and *Zblast SD+*).

## What is left out, on purpose

- **The post texts.** Look posts up by Message-ID in a Usenet archive, such as the archive.org copies of the group or a news server that still carries it. (Google Groups no longer finds posts by Message-ID.)
- **E-mail addresses.** Poster names are kept as written, with any address removed; 795 posts give no name. 139 Message-IDs had been built from the poster's address by their newsreader; there the address part is replaced by `…`, so those posts have to be found by date and subject instead.
- **Guesses about who is who.** Which addresses belong to one person, and which poster may be which ZXDB label, were worked out by rules that are sometimes wrong, so they are not included.
- **Spam.** None of these posts is spam.

## Examples

```sql
-- everything linked to one ZXDB entry (3012 = Manic Miner)
SELECT date, reftype, subject, score, extra FROM v_postrefs WHERE entry_id = 3012 ORDER BY date;

-- who sent the Tipshop what for Manic Miner
SELECT date, kind, person FROM v_tipshop_credits WHERE entry_id = 3012 ORDER BY date;

-- CSSCGC posts per contest year
SELECT tag_name, count(*) FROM v_postrefs WHERE tag_id IS NOT NULL GROUP BY tag_name;

-- release announcements of ZXDB entries, first announcement first
SELECT min(p.date) AS first, e.title, count(*) AS posts
FROM postrefs r JOIN posts p ON p.id = r.post_id JOIN zxdb_entries e ON e.id = r.entry_id
WHERE r.referencetype_id = 7 AND r.entry_id IS NOT NULL AND r.score >= 0.5
GROUP BY e.id ORDER BY first;

-- the history of the TZX format, oldest first
SELECT t.first_date, t.subject, tt.verdict, tt.summary
FROM topic_threads tt JOIN threads t ON t.id = tt.thread_id WHERE tt.topic = 'tzx' ORDER BY t.first_date;
```

To join with ZXDB itself: `ATTACH 'zxdb.sqlite' AS zx`, then join `entry_id` to `zx.entries.id`, `label_id` to `zx.labels.id` and `tag_id` to `zx.tags.id`.
