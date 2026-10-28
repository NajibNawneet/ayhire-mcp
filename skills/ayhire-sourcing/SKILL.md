---
name: ayhire-sourcing
description: Use when the user wants to hire or source for a role ("find me a senior Rust engineer", "fill this role", "who should I talk to for our AI agents job?"), including roles that are only partly technical (solutions, sales engineering, developer relations, AI deployment or strategy, GTM engineering), to see who to talk to first, or to manage their roles and shortlist in AyHire. A playbook for creating a role, searching from several angles, choosing who to add and why, checking the evidence, and presenting picks.
---

# Sourcing a role with AyHire

AyHire gives you the parts; you decide how to use them, like a good recruiter browsing GitHub would: try a search, see it's off, try another angle, go back. Roles and the shortlist are shared with the AyHire web app: what you add or shortlist here shows up there, and the reverse.

## 1. Understand the role

Get the essentials from what the user said: what the person will build, the stack, seniority, and any hard constraints (location or time zone, must be open to work). Ask one short question only if something essential is missing; otherwise go with sensible defaults and say what you assumed.

### Roles that aren't pure engineering: don't refuse, look

Some roles are mostly sales, advising, teaching or strategy with some technical depth (an AI deployment strategist who is 80% sales, a solutions engineer, developer relations). Never decline them as "not a GitHub role". Instead:

1. Say in one or two sentences what GitHub can and can't show for this role. It can show how someone thinks and writes (playbooks, adoption guides, decision write-ups, workshop and talk material), whether they build or can defend what they recommend, and what they say about their own client work. It can't show selling: pipeline, deals, references, how they come across in a room. That part has to be checked off GitHub.
2. Then look for real public evidence anyway:
   - `search_people` with words people put in their bios for this kind of work ("AI adoption", "solutions architect", "forward deployed", "customer engineer", "developer advocate", "AI strategy"), with the location if there is one;
   - `search_repos` by topic for the kind of material they publish (for example `ai-adoption`, `ai-strategy`, `ai-transformation`, `ai-governance`, `ai-enablement`, `playbook`, `agent-skills`), then the people behind the best of those repos (`find_people_behind_repos` with `repo` or `org`);
   - the organizations whose deployment, solutions or enablement teams publish material (their org repos, then their people);
   - `get_person_evidence` for each promising person: the profile README is often where people describe client-facing work, and `repo_counts.prose` shows how much non-code material they publish; check the commit history shape (many days of iteration vs one bulk upload) to tell a practitioner from a content dump.
3. Present what you found and say plainly where the evidence is thin, which candidates rest mostly on what they say about themselves, and what to verify off GitHub. A short honest list beats an empty one.

## 2. Create the role, or reuse one

- If the user hires for more than one company, make sure you're in the right workspace: `list_workspaces`, then `switch_workspace` or pass `workspace` (or `create_workspace` for a new company).
- `list_roles` first if the user may already have this role.
- `create_role` with a clear title and a one-paragraph brief. AyHire suggests the role's criteria from them (what the role needs, each with an importance: must, important or nice). Show the criteria to the user in a few words and change them with `set_role_criteria` if they don't match what the user needs; people are judged against them. `suggest_role_criteria` proposes a fresh set when the brief changes.

## 3. Plan several angles before you search

One keyword search is rarely enough: GitHub search matches strings, not people ("gtm" is also Google Tag Manager; "compaction" now mostly means LLM context compaction). Before searching, write down 3 to 5 angles, for example:
- **Seed repos**: the projects this person would work on or the community lists of them (`search_repos` by topic is cleaner than keywords; no star floor by default). Then mine each good seed for its people with `find_people_behind_repos(repo=...)` or `org=...`. For engineers this is usually the strongest angle.
- **Profiles**: `search_people` matches name, login and bio, with `location` and `language`. Best when profiles say more than repos, and for a location constraint.
- **The work itself**: `search_commits` for phrases only practitioners write ("smartlead webhook", "mvcc snapshot isolation").
- **Topics and languages**: `find_people_behind_repos(query/topic, language, location)`, then `page: 2` and on for the next repos.

## 4. Search, with the role attached

Pass `role_id` to every search. Each search is then logged in the role's history (the user can see what ran), everyone it returns is marked as seen, and people the role has already seen are left out, so each round brings new people. Pass `include_seen: true` only when you deliberately want them back.

- Read what comes back. If it's off-target (wrong field, toys, AI-generated repos, the wrong meaning of a word), say so to yourself and change the angle rather than pressing on.
- Use `location` when the role has one: countries, cities or regions (`Europe`, `US`, `North America`, `UK` also match their countries and main cities). It matches what people wrote on their profile. People who list no location are kept and marked `location_match: "unknown"`: their location is unknown, not wrong, so ask them (pass `location_unknown: "exclude"` only if the user wants confirmed locations only). Never rank anyone by where they live, and never guess it (or a time zone) from commit times.
- `search_people` needs every word to match, so keep each query to one to three words and run more searches rather than longer ones.
- Go past the first page when the angle is good: `page` (repos) or `cursor` (people). Aim for a real pool, not the first 30 names.
- The role's own search (`find_people_for_role`) still works: it adds everyone it finds. Prefer choosing people yourself.

## 5. Choose who to add, and say why

From what the searches showed (bios, top repos, the repos they build and how much), pick the people who actually fit. Skip organizations, bots, people with a handful of contributions, and copies of other people's work.

Searches only mark people as seen; add the ones you choose with `add_people_to_role`, each with a one-line reason drawn from their work, for example "Wrote the MVCC layer of tursodatabase/turso (103 merged PRs titled mvcc)". The role keeps the reason and how they were found. Add in batches of up to 25.

## 6. Check the evidence

Before you score or present anyone, look at the work: `get_person_evidence` (with `role_id`) shows their repos, how they worked on them, their work on other people's projects, PRs and orgs, all linked. For a big project, `contribution_profile(username, repo)` shows their merged PRs there and the code areas they touched. Read the README of the project that matters (`read_readme`). Drop anyone the evidence doesn't support; note what is unknown (unknown is not a weakness).

## 7. Score the ones worth it

`score_role_person` analyzes one person (about 20 seconds when new, counting toward 20 an hour; saved analyses are free; it adds them to the role if they aren't in it yet) and judges them against the role's criteria: a verdict per criterion (strong, some, unknown, none), a reason, the links it rests on, and interview questions for what the evidence leaves open. Unknown is never a negative; only clear evidence against a must-have sets someone aside. Score the people you'd present, not everyone. If it returns `limited`, stop and tell the user when more analyses are available (`reset_at`). If a call times out, call it once more for that person (the analysis usually finished), but don't retry in a loop. Total scores are out of 40. People marked `needs_rescore` were scored the old way (their own repos only): score them again before you judge them.

## 8. Present the picks

Call `get_role` once you've finished scoring: each `score_role_person` result shows the order at that moment, and it changes as you score more people. Lead with the answer:

- The top 3 to 5 picks: name with a GitHub link, why they fit the brief from what they actually built (with a link to the evidence), availability, and the one thing to check.
- Present them in `get_role`'s order: people judged against the criteria come first, ordered by the evidence of fit (`ranking_basis`), not by total score (the total is general, not fit). Quote their verdicts (`fit.verdicts`) with a link each, say what is unknown, and pass on the interview questions. People not judged yet are in the general order: judge them before presenting them as a fit.
- Then one line on who was set aside, and why (the public evidence points away from a must-have, with its link).
- Write about people the way the `ayhire` skill says: missing evidence is "nothing public shows X", never "has no X"; keep AyHire's "likely" and "guess"; no location or time zone that isn't on their profile; names or "they", never a guessed gender.
- Being employed is not a reason to skip someone. Most strong engineers have a job, so say "employed at X" and let the user decide. For someone strong who has a job, offer to watch them (`watch_person`) so the user hears when they say they're open.
- Say how you searched (the angles, how many people you looked at) so the user can judge coverage.

## 9. Act on the user's decisions, and the next round

- `decide` with `shortlisted` or `not_now`, plus a short note in the user's words if they gave a reason.
- `get_shortlist` shows everyone shortlisted, across roles.
- Look up contact details (`enrich_person`) only for people the user wants to approach: it's a paid lookup. People in a role come back with `email` when AyHire already has one.
- For more people later, search again with `role_id`: people already seen are left out automatically, and `get_role` shows `recent_searches` so you don't repeat an angle.
