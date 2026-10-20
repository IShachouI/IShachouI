# Including anonymized private contributions in the snake

By default the snake animation only reflects **public** contributions, because the
GitHub Actions `GITHUB_TOKEN` runs as the `github-actions[bot]` identity — a
non-owner — and GitHub's GraphQL API returns anonymized private contribution
counts **only to your own account's token**. Enabling the profile setting alone
is *not* enough for the Action; it needs a token that authenticates as you.

Only **daily contribution counts** are ever read (via
`user.contributionsCollection.contributionCalendar`). Repository names, code,
commit messages, and all other private metadata are **never** fetched, and no
repository's private status is changed.

## One-time setup (≈2 minutes)

1. **Enable private contributions on your profile** (so your native profile graph
   matches the snake):
   GitHub → **Settings → Public profile → Contributions** →
   check **"Include private contributions on my profile."**

2. **Create a minimal, read-only token** — GitHub → **Settings → Developer
   settings → Personal access tokens → Tokens (classic) → Generate new token
   (classic)**:
   - Note: `snake-contribution-calendar`
   - Expiration: your choice (shorter is safer; remember to rotate)
   - Scopes: select **only `read:user`** (read-only profile/user data — no
     repository access)

   > `read:user` is the required scope — it is all that is needed to read your
   > own contribution calendar (daily counts), and it cannot read any repository
   > contents. Do **not** grant broader scopes such as `repo`.

3. **Add it as a repository secret** — repo → **Settings → Secrets and variables
   → Actions → New repository secret**:
   - Name: `SNK_GITHUB_TOKEN`
   - Value: the token from step 2

   GitHub stores it encrypted and masks it in all Action logs.

4. **Run the workflow** — **Actions → Generate Snake Animation → Run workflow**
   (or wait for the daily 00:00 UTC run). The generated SVGs on the `output`
   branch will now include your anonymized private contributions.

## Verifying

- Compare the number of filled squares before/after the first run with the
  secret set — it should jump from public-only to the full (private-inclusive)
  graph.
- Your account-level total is visible only to you via:
  `gh api graphql -f query='{ user(login:"IShachouI"){ contributionsCollection { contributionCalendar { totalContributions } restrictedContributionsCount } } }'`

## Rotating an expired or compromised token

Classic PATs expire. When the token expires (or if you want to replace it), the
workflow keeps running but silently falls back to public-only contributions
until you supply a fresh token:

1. Create a new classic PAT with **only `read:user`** (same as step 2 above).
2. Update the secret in place — repo → **Settings → Secrets and variables →
   Actions → `SNK_GITHUB_TOKEN` → Update secret** — and paste the new value
   (the secret name stays the same, so no workflow change is needed).
3. Re-run the workflow — **Actions → Generate Snake Animation → Run workflow** —
   so the SVGs are regenerated with the new token.
4. Delete the old token under **Settings → Developer settings → Personal access
   tokens** so it can no longer be used.

## Security notes

- The token is read-only (`read:user`) and lives only as an encrypted Actions
  secret — never in the README, the SVGs, the source, or the logs.
- Removing the `SNK_GITHUB_TOKEN` secret does **not** immediately strip private
  contribution counts from the SVGs already published on the `output` branch —
  those files stay as-is until the workflow **runs again successfully** with the
  default token, at which point they are replaced with the public-only graph.
  To force that immediately, trigger a run via **Actions → Generate Snake
  Animation → Run workflow** after removing the secret.
