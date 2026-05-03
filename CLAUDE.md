# Really Good Trades — Claude Working Instructions

## Project

Single-file static site. Main files: `index.html`, `profile.html`, 
`admin.html`, `sets/*.js`. Deployed via Netlify from GitHub. 
Supabase for auth/data.

## Versioning — mandatory on every edit

- Format: `v1.0.[build].[MM].[DD]`
- Build increments by 1 each change (31 → 32 etc.)
- Update BOTH: line 1 comment AND the footer `<span>` in index.html
- Current: v1.0.31.05.01

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