# Really Good Trades — Claude Working Instructions

## Project

Single-file static site. Main files: `index.html`, `profile.html`, 
`admin.html`, `sets/*.js`. Deployed via Netlify from GitHub. 
Supabase for auth/data.

## Versioning — automated

- Format: `v1.0.[build].[MM].[DD]`
- A PostToolUse hook in `.claude/settings.json` auto-bumps the version in `index.html` after every file edit in this project — do NOT bump it manually
- Current: v1.0.33.05.04

## After every change, provide

A git commit command in this format:

git add [files changed]
git commit -m "[description of what changed and why]"
git push

## Auth / logout bug history

- v1.0.30: Boot session check timeout raised 5s→10s, stopped wiping 
  localStorage on timeout
- v1.0.31: loadUserList timeout was wrongly calling emergencyLogout() 
  — fixed to go offline instead
- Sentry is live for auth event logging
- Admin auth uses isolated storageKey `sb-rgt-admin`
- Random logout leading theory (admin tab + shared localStorage) 
  may be resolved — monitoring via Sentry

## Stack notes

- Card data lives in `sets/*.js`, registered via `registerSet()`
- Supabase client is `_sb` throughout
- `emergencyLogout()` is the nuclear option — only call it on 
  genuine auth failure, never on network timeouts

## Adding new card sets

Always do this via Claude Code (not claude.ai chat) for consistency.

Workflow:
1. Open Claude Code in the repo root
2. Paste raw set data and say "convert this into a set file matching 
   the existing format in sets/"
3. Claude Code reads existing sets/ files to match format exactly
4. Review the generated file, then commit and push

- One file per set: `sets/[setname].js`
- Must use `registerSet()` — match the pattern in existing set files
- Never create set files in claude.ai chat

## Preview before pushing to main

### Set files (sets/*.js)
- Open `index.html` directly in Chrome from File Explorer
- Check the set displays correctly
- Then commit and push to main

### HTML / auth changes (index.html, admin.html, profile.html)
1. `git checkout -b preview-[description]`
2. Commit changes and push the branch
3. Netlify generates a preview URL: `preview-[description]--your-site.netlify.app`
4. Test against real Supabase environment
5. Merge to main when happy
- Branch deploys are enabled in netlify.toml and Netlify dashboard