---
name: ayhire
description: Use for any question about people as they show up in their public GitHub work, answered from real GitHub data instead of memory or web search. Who builds, maintains or contributes to an open-source project or a kind of project ("who is behind the popular AI agent frameworks?", "core contributors to Rust databases"), whether someone is a good engineer ("is @someone any good?"), comparing or shortlisting people, and sourcing candidates for a role, including roles that are only partly technical (solutions, sales engineering, developer relations, AI deployment or strategy) where public work is playbooks, guides, workshops or skills. Covers the AyHire tools, how to present results, and what to do when AyHire isn't connected.
---

# AyHire

AyHire evaluates people from what they actually built and published on GitHub (repos, READMEs, commits, pull requests, activity), not from CVs. That covers engineers, and also people in partly technical roles whose public work is playbooks, guides, workshop material or skills. You reach it through the `ayhire` MCP tools.

## If the tools are missing or fail with an auth error

The AyHire tools need a one-time sign-in. Run `~/.local/bin/ayhire status`. If it exits with 3, run `~/.local/bin/ayhire login` with a long timeout (it waits up to 10 minutes while the user signs in through the browser), then ask the user to run `/mcp` and reconnect `ayhire`. Then continue with the user's request. Never ask the user to paste an API key into the chat.

## If the user wants to sign out, disconnect or forget AyHire

Never sign out by deleting `~/.config/ayhire/credentials.json` or any other file. The key keeps working on AyHire until it is revoked, and a connection that is already open keeps using it.

1. Call the `disconnect` tool. It ends this agent's connection on AyHire's side straight away.
2. If AyHire was set up with the `ayhire` command line on this computer, also run `~/.local/bin/ayhire logout; echo "exit_code=$?"`. It deletes the saved key (after `disconnect`, it says the key was already revoked). `0`: done. `5`: AyHire couldn't confirm the key was revoked and it is still saved: tell the user, and that they can revoke it at https://ayhire.com/app/integrations#keys.
3. Tell the user it's done and that AyHire's tools stop working here from the next call. If AyHire was added to other agents with the same key, it stops working there too.

After a disconnect, don't sign in again unless the user asks to reconnect.

## Tools

| Tool | Use it for |
|---|---|
| `analyze_github_user` | One specific person. Returns 4 general scores (0-10, total out of 40), a summary, top skills, red flags, `outside_work` (their commits and PRs on other people's repos, last year) and `activity` (their GitHub contribution calendar for the last year: total, active weeks, streaks, last active). A saved result from the last 30 days comes back instantly and free (`cached: true`). A new analysis takes 15-25 seconds. |
| `analyze_github_users` | Several people at once (up to 25), ranked by total. Saved ones are free; up to 4 new ones per call, the rest come back in `pending`: call again with those. |
| `score_candidate` | Same scores without saving the person. Use only when the user doesn't want them stored. |
| `find_people_behind_repos` | Find who builds open source: top repos by stars for a query, topic or language, then their main contributors, ranked by all-time contributions (so early contributors rank high; `list_commits` or `contribution_profile` shows who works there now). Pass `repo` ("owner/name") for one exact project, or `org` ("vercel") for an organization's repos; `public_member: false` only means they aren't shown as a member. `location` keeps people whose GitHub location matches a country, city or region (Europe, US, North America, UK), plus people who list none, marked `location_match: "unknown"` (`location_unknown: "exclude"` leaves them out), `page` reads the next repos, `created_after` / `pushed_after` filter by date. With `role_id`, people the role has seen are skipped and the search is logged; it doesn't add them to the role. |
| `enrich_person` | Email, LinkedIn (URL, headline, current companies) and X for one person, each with its source and confidence. Usually $0.001-0.015, free when saved. |
| `check_availability` | "Already hired?" A cautious verdict (employed, employed_but_open, open_to_work, likely_employed, unknown) with the evidence. |
| `search_candidates` | Plain-language search over people AyHire has already analyzed. It only finds people analyzed before. |
| `get_person_evidence` | The evidence a recruiter would click through on GitHub, in one call, every item linked: profile and profile README, repos (code, prose, fork) with descriptions, how they worked on their top repos (commits, active days, AI-assisted commits), work on other people's projects (last year), PRs to other repos with titles and merge counts, orgs, activity by month, and with `role_id` the repos the role found them through. Free, live, about 5 s. |
| `contribution_profile` | What someone did in one repo: merged PRs (count, first and last, newest with links), the code areas they touched (e.g. `core/mvcc`), PRs they reviewed, their commits and contributor rank. Checks "core contributor to X". |
| `read_readme` | The whole README of any public repo. Read the work before you judge it. |
| `list_pull_requests` / `list_commits` | Every PR a person opened (by repo or state), or their commits in one repo, with links, co-authors and AI-assisted flags. |
| `search_people` | Search GitHub profiles (name, login, bio) with `location`, `language`, follower and repo minimums. GitHub matches every word, so use one to three words per search. Each person comes with bio, location, company, followers and top repos. |
| `search_repos` | Search repos by words, topic, language, org, stars and dates (no star floor by default): seed lists and projects to mine for people. |
| `search_commits` | Who wrote commits mentioning something ("hubspot sync"), grouped by person with example commits. |
| `create_role` | Start sourcing for an open position. AyHire turns the title and brief into a GitHub search unless you pass `query` / `language`. |
| `set_role_criteria` / `suggest_role_criteria` | What the role needs: criteria with an importance (must, important, nice), judged from public work. `create_role` suggests them; change them with the user's words. |
| `find_people_for_role` | Run the role's own search and save everyone it finds to the role. Each round leaves out people the role has already seen; `page` for the next repos. |
| `add_people_to_role` | Add the people you chose (from any search) to a role, each with a one-line reason. The role keeps the reason and how they were found. |
| `score_role_person` | Analyze one person, judge them against the role's criteria and check availability, adding them to the role first if they aren't in it. You choose who is worth it. The order changes as you score more people: `get_role` gives the final one. |
| `get_role` | A role with its criteria, ranked with AyHire's call per person: people judged against the criteria first (by evidence of fit; `fit` has a verdict, reason and links per criterion, and interview questions), then the rest in the general order (`ranking_basis` says which). |
| `decide` | Shortlist someone (`shortlisted`), set them aside (`not_now`), or undo (`none`), with an optional note. |
| `get_shortlist` | Everyone the user shortlisted, across roles. Same list as the web app's Shortlist. |
| `watch_person` / `unwatch_person` / `list_watches` | Watch someone (with or without a role) and choose what to hear about; AyHire re-checks their public GitHub every day and shows changes on Home; `list_watches` also names open roles each may fit (`may_fit`). Ask before `paid_checks: true` (a paid LinkedIn re-check every 2 weeks). |
| `list_roles` | The roles in the current workspace, with counts. |
| `list_workspaces` / `switch_workspace` / `create_workspace` | Workspaces hold roles and their shortlist: one per company the user hires for, shared with that company's team. Role tools use the current one (the same as the web app) unless you pass `workspace`. |
| `list_workspace_members` / `invite_to_workspace` | Who is in a workspace, and inviting a teammate by email (only into a company workspace the user owns, not Personal). The invite emails a join link and returns it; it works for 7 days for that address only. Confirm the address with the user before inviting. |
| `whoami` | Check the connection and which account is signed in. |

Rules:
- Check before you claim. Analyses return `findings` and `red_flags`, each with the `sources` (GitHub links) it rests on; AyHire drops any claim it can't back. Quote them with their links. For anything else you say about someone, look at `get_person_evidence` (and `read_readme` for the work that matters) and back it with a link. If the evidence doesn't support a claim, don't make it. `unknown` means GitHub couldn't show it this time, not that it's missing.
- Don't pass `refresh: true` unless the user asks for a fresh analysis. Saved results are free; new ones count toward a limit of 20 per hour.
- To compare several people, use `analyze_github_users` with all their usernames, and call it again with `pending` until it's empty.
- `find_people_behind_repos` returns each person's real name, company, location and GitHub `hireable` flag. Use those instead of guessing names. A company is a hint that they're employed; it's not proof.
- Sourcing for a real opening: follow the `ayhire-sourcing` skill (create the role, search from several angles with `role_id`, add the people you choose with reasons, check the evidence, score, present). For a quick one-off question ("who builds X?"), `find_people_behind_repos` is enough. Don't analyze everyone in a list.
- Roles and the shortlist are shared with the AyHire web app: what you shortlist here shows up there, and the reverse.
- GitHub usernames only. If the user gives a LinkedIn URL, ask for the GitHub username (LinkedIn to GitHub isn't supported yet).
- Don't turn a role away because it isn't pure engineering. Say what GitHub can and can't show for it, then search anyway (the `sourcing` skill says how) and be honest when the evidence is thin.
- `enrich_person` and `check_availability` spend a shared prepaid balance. Use them on the people the user is seriously considering, not on every name in a list. Only pass `deep: true` when the normal lookup found no LinkedIn and the user wants more.
- Present contact data with its source: a LinkedIn with `linkedin_confidence: "guess"` is a name match, so say it may be the wrong person. Never present a guess as fact.
- People in a role come with `email` when AyHire has one (`address`, `source`: github_profile or lookup, `confidence`: confirmed, likely or guess). Show it with its source and confidence. AyHire never sends email; don't offer to send it from AyHire.

## Presenting results

- Lead with the answer to the user's question, then the evidence.
- Name each person with a link to their profile: `[@username](https://github.com/username)`.
- Show the four scores compactly. For several people, use one table sorted by the total (out of 40), with columns: depth, shipping, communication, gem.
- The four scores are general. They describe someone's public work, not their fit for a particular role. Fit to a role comes from its criteria (`fit` in role tools): quote those verdicts with their links. Never present a total as "fit for the role".
- Include the red flags. Don't soften them.
- Don't invent facts the tools didn't return (emails, LinkedIn, salary, availability). If a lookup found nothing, say so.
- Never rank or judge people by where they live. Use location only when the user sets it as a requirement, and treat an unknown location as unknown.

How to write about a person:
- Missing evidence is unknown. Write "nothing public shows X", never "has no X" or "lacks X", unless AyHire's verdict is `none` with the evidence linked.
- Keep AyHire's confidence words: "likely employed at X", an email that is "likely" or "a guess". Don't turn a likely into a fact.
- Where someone lives comes only from what they wrote on their profile. Never infer a location or time zone from commit times, names or languages. When location matters and is unknown, say "location unknown, ask them".
- Refer to people by name, or as "they". Don't guess anyone's gender.
- Don't pass on someone you haven't scored unless you link what rules them out. Otherwise list them as not checked yet.

What the scores mean:
- **technical_depth**: how substantial and widely used their work is, on their own repos and on other people's projects
- **shipping_velocity**: how often and how recently they ship
- **communication**: README, commit message and pull request quality
- **gem_score**: strong work with little public visibility (a hidden gem); a low one only means people already know them

## Not available yet

Going from a LinkedIn profile to a GitHub account, and outreach (sending messages). If the user asks for one of these, say so in one line and offer what works today.
