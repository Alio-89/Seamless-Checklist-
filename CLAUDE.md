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