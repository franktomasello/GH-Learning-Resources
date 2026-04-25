# 🔄 SVN to GitHub Migration Runbook

> **Complete guide to converting Subversion (SVN) repositories to Git and pushing them to GitHub**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the commands:**

- **Extract SVN authors:** `svn log --quiet URL | awk '/^r/ {print $3}' | sort -u > svn-authors-raw.txt`
- **Convert with svn2git:** `svn2git https://svn.example.com/repo --authors authors.txt --verbose`
- **Track large files:** `git lfs track "*.zip" && git lfs migrate import --include="*.zip" --everything`
- **Push to GitHub:** `git push --all origin && git push --tags origin`

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| `svn2git` or `git svn` installed on migration machine | ☐ |
| Network access to SVN server | ☐ |
| Authors mapping file (`authors.txt`) prepared | ☐ |
| Git LFS installed (`git lfs install`) for large binary handling | ☐ |
| Target GitHub repository created | ☐ |

---

## 📋 Overview

| Step | Tool | Purpose |
|------|------|---------|
| **Author mapping** | Manual text file | Map SVN usernames to Git name + email |
| **SVN-to-Git conversion** | `svn2git` or `git svn` | Convert history, branches, and tags |
| **Large binary handling** | `git lfs` | Track binaries with Git LFS to avoid bloat |
| **Push to GitHub** | `git push` | Upload the converted repo |
| **Validation** | Manual checklist | Confirm history, branches, and tags are correct |

---

## 1️⃣ Create the Authors File

*Maps SVN usernames to Git author identities. Every SVN committer must have a mapping or the conversion will fail.*

### Generate an Authors List from SVN

```bash
svn log --quiet https://svn.example.com/repo | \
  awk '/^r/ {print $3}' | \
  sort -u > svn-authors-raw.txt
```

### Create the Authors Mapping File

Create a file called `authors.txt` with the following format:

```
jsmith = John Smith <jsmith@example.com>
adoe = Alice Doe <adoe@example.com>
ci-bot = CI Bot <ci-bot@example.com>
(no author) = Unknown <unknown@example.com>
```

> 💡 **Tip:** The `(no author)` entry handles commits with no SVN username. Every username from `svn-authors-raw.txt` must have a corresponding line in `authors.txt`.

---

## 2️⃣ Convert SVN to Git Using svn2git (Recommended)

### Install svn2git

```bash
# macOS
brew install svn2git

# Ubuntu/Debian
sudo apt-get install git-svn ruby
sudo gem install svn2git
```

### Standard Layout Conversion (trunk/branches/tags)

```bash
svn2git https://svn.example.com/repo \
  --authors authors.txt \
  --verbose
```

> ✅ **Result:** SVN trunk becomes `main` (or `master`), SVN branches become Git branches, and SVN tags become Git tags.

### Non-Standard Layout

If the SVN repo does not use the standard `trunk/branches/tags` layout:

```bash
# Trunk only (no branches or tags)
svn2git https://svn.example.com/repo \
  --authors authors.txt \
  --rootistrunk

# Custom directory names
svn2git https://svn.example.com/repo \
  --authors authors.txt \
  --trunk main-line \
  --branches feature-branches \
  --tags release-tags
```

---

## 3️⃣ Alternative: git svn clone (More Control)

*Use `git svn` when you need more control over the conversion process or when svn2git encounters issues.*

### Clone with Full History

```bash
git svn clone https://svn.example.com/repo \
  --stdlayout \
  --authors-file=authors.txt \
  --no-metadata \
  --prefix="" \
  my-repo
```

| Flag | Purpose |
|------|---------|
| `--stdlayout` | Assumes standard `trunk/branches/tags` layout |
| `--authors-file` | Maps SVN users to Git identities |
| `--no-metadata` | Omits `git-svn-id` lines from commit messages |
| `--prefix=""` | Avoids `origin/` prefix on remote-tracking branches |

### Convert SVN Branches and Tags to Proper Git Refs

After `git svn clone`, SVN branches appear as remote refs. Convert them:

```bash
cd my-repo

# Convert remote branches to local branches
for branch in $(git branch -r | grep -v 'tags/' | grep -v 'trunk'); do
  git branch "$(echo $branch | sed 's/^ *//')" "refs/remotes/$branch"
done

# Convert SVN tags to proper Git tags
for tag in $(git branch -r | grep 'tags/'); do
  tagname=$(echo $tag | sed 's|^ *tags/||')
  git tag "$tagname" "refs/remotes/$tag"
done

# Remove the SVN remote tracking refs
git branch -r -d $(git branch -r)
```

---

## 4️⃣ Handle Large Binaries with Git LFS

> ⚠️ **Warning:** SVN repositories often contain large binaries (JARs, ZIPs, DLLs, etc.) that will bloat the Git repo. Migrate these to Git LFS before pushing to GitHub.

### Install Git LFS

```bash
git lfs install
```

### Track Large File Types

```bash
git lfs track "*.zip"
git lfs track "*.jar"
git lfs track "*.dll"
git lfs track "*.war"
git lfs track "*.ear"
git lfs track "*.exe"
git lfs track "*.iso"
```

### Migrate Existing History (Optional but Recommended)

```bash
git lfs migrate import \
  --include="*.zip,*.jar,*.dll,*.war,*.ear,*.exe,*.iso" \
  --everything
```

> 💡 **Tip:** The `--everything` flag rewrites all branches and tags. This changes commit SHAs but ensures large files are tracked by LFS throughout the entire history.

### Verify LFS Tracking

```bash
git lfs ls-files
```

---

## 5️⃣ Convert svn:ignore to .gitignore

### Export svn:ignore Properties

```bash
# From within the working copy (before conversion) or from the SVN repo
svn propget svn:ignore -R https://svn.example.com/repo > svn-ignores.txt
```

### Create .gitignore

Translate the SVN ignore patterns to `.gitignore` format:

```bash
# Example .gitignore based on common SVN ignores
target/
*.class
*.jar
*.war
*.log
.settings/
.project
.classpath
bin/
build/
```

> 💡 **Tip:** SVN ignore properties are per-directory. Git uses a single `.gitignore` file (with optional subdirectory overrides). Consolidate all SVN ignores into one `.gitignore` at the repo root.

After creating `.gitignore`, commit it:

```bash
git add .gitignore
git commit -m "Add .gitignore converted from svn:ignore properties"
```

---

## 6️⃣ Push to GitHub

### Create the Target Repository

```bash
gh repo create MyOrg/my-repo --private --confirm
```

### Add Remote and Push

```bash
# Set the remote
git remote add origin https://github.com/MyOrg/my-repo.git

# Push all branches
git push --all origin

# Push all tags
git push --tags origin
```

> ✅ **Result:** All Git history, branches, and tags are now on GitHub.

---

## 7️⃣ SVN-to-Git Structure Mapping

| SVN Concept | Git Equivalent |
|-------------|---------------|
| `trunk/` | `main` branch (default branch) |
| `branches/feature-x` | `feature-x` branch |
| `tags/release-1.0` | `release-1.0` tag |
| `svn:ignore` | `.gitignore` |
| `svn:externals` | Git submodules or subtrees |
| Revision numbers (r1234) | Commit SHAs (a1b2c3d) |
| `svn update` | `git pull` |
| `svn commit` | `git add` + `git commit` + `git push` |
| `svn checkout` | `git clone` |

> 💡 **Tip:** Share this mapping table with your team during the transition period. Developers used to SVN will find the mapping helpful.

---

## 8️⃣ Validation Checklist Post-Migration

| Check | Command | Expected Result |
|-------|---------|-----------------|
| **Commit count** | `git rev-list --count HEAD` | Matches SVN revision count (approximately) |
| **Branch list** | `git branch -a` | All SVN branches present |
| **Tag list** | `git tag -l` | All SVN tags present |
| **First commit** | `git log --reverse --oneline \| head -1` | Matches earliest SVN commit |
| **Latest commit** | `git log --oneline -1` | Matches latest SVN commit |
| **Author mapping** | `git log --format='%aN <%aE>' \| sort -u` | No unmapped SVN usernames |
| **LFS files** | `git lfs ls-files` | Large binaries tracked by LFS |
| **File content** | Compare key files manually | Content matches SVN HEAD |
| **Build / CI** | Run the existing build process | Build succeeds on the Git repo |

---

## 9️⃣ Tips for a Smooth Transition

### Keep SVN Read-Only for 2-4 Weeks

| Phase | SVN State | Git State | Duration |
|-------|-----------|-----------|----------|
| **Pre-migration** | Read/write | Does not exist | -- |
| **Migration** | Read-only | Receiving pushes | 1-2 days |
| **Transition** | Read-only (reference) | Active development | 2-4 weeks |
| **Decommission** | Archived/offline | Primary | After transition |

> 💡 **Tip:** Set SVN to read-only immediately after the final migration. This prevents commits that would be lost. Keep it available for reference during the transition period.

### Provide an SVN-to-Git Cheat Sheet for Your Team

| SVN Command | Git Equivalent |
|-------------|---------------|
| `svn checkout URL` | `git clone URL` |
| `svn update` | `git pull` |
| `svn status` | `git status` |
| `svn diff` | `git diff` |
| `svn add FILE` | `git add FILE` |
| `svn commit -m "msg"` | `git commit -m "msg"` then `git push` |
| `svn log` | `git log` |
| `svn revert FILE` | `git checkout -- FILE` |
| `svn switch BRANCH` | `git checkout BRANCH` |
| `svn merge` | `git merge` |
| `svn blame FILE` | `git blame FILE` |

> 💡 **Tip:** Post this cheat sheet in your team wiki or Slack channel. The biggest adjustment for SVN users is that Git separates `commit` (local) from `push` (remote).

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Author mapping error** | An SVN username in history is missing from the authors file, including bot or no-author commits. | Regenerate the unique author list from SVN logs, map every value exactly once, and rerun the conversion from a clean output directory. |
| **Branches or tags are missing after conversion** | The SVN repository does not use the expected trunk/branches/tags layout or has nested project paths. | Run an SVN layout inventory, pass the correct trunk/branches/tags paths to the converter, and validate refs before pushing to GitHub. |
| **Push rejected because private email would be published** | GitHub account settings block pushes that expose a private email address. | Use a GitHub noreply email in rewritten commits or temporarily adjust the account email privacy setting for the migration account. |
| **Large files or LFS objects fail to push** | The converted Git repository contains files above GitHub size guidance or missing LFS migration. | Run a large-file scan, migrate binaries to Git LFS where appropriate, and push LFS objects before final validation. |

---

## ❓ Common Questions & Troubleshooting

### Q: `svn2git` fails because our SVN repo does not use the standard trunk/branches/tags layout. What should I do?
**A:** Use `--rootistrunk` if the root of the SVN repo is the trunk (no branches or tags directories). For custom directory names, specify them explicitly with `--trunk`, `--branches`, and `--tags` flags. If the layout is highly non-standard, try `--no-minimize-url` or switch to `git svn clone` with manual path specifications for more control.

---

### Q: The conversion fails with author mapping errors. How do I fix this?
**A:** Every SVN username that appears in the repository history must have a corresponding entry in the `authors.txt` file. Run the SVN log extraction command to get all unique usernames, then verify each one has a mapping. Common issues include missing entries for `(no author)` (commits with no SVN username), bot accounts, and usernames with special characters.

---

### Q: The SVN-to-Git conversion is taking many hours for our large repository. Is there a faster approach?
**A:** For very large repositories, use `git svn clone` with incremental fetching rather than converting everything at once. You can also clone a subset of history using `--revision START:HEAD` to skip ancient history. Another option is to split the SVN repo into smaller logical units and convert them into separate Git repos. Running the conversion on a machine with fast network access to the SVN server also helps significantly.

---

### Q: GitHub is rejecting our push because of files over 100MB. How do we handle large binaries?
**A:** Files over 100MB must be tracked with Git LFS before pushing to GitHub. Run `git lfs track "*.ext"` for each large file type, then use `git lfs migrate import --include="*.zip,*.jar,*.dll" --everything` to rewrite history so large files are stored in LFS throughout. Push again after the LFS migration. Note that this rewrites commit SHAs.

---

### Q: The `.gitignore` generated from `svn:ignore` properties does not seem correct. What should I check?
**A:** `git svn show-ignore` outputs SVN ignore properties in a format similar to `.gitignore`, but SVN ignores are per-directory while Git uses a single file (with optional subdirectory overrides). Manually verify the output and consolidate patterns into a root `.gitignore`. Pay attention to path prefixes -- SVN ignore patterns are relative to each directory, while `.gitignore` patterns are relative to the repo root unless prefixed with `/`.

---

### Q: After conversion, some SVN branches or tags are missing from the Git repo. What went wrong?
**A:** Check that `svn2git` or `git svn` successfully processed all branches. Empty branches (with no unique commits) may not convert. Also verify the SVN branch/tag paths match what the conversion tool expects. After `git svn clone`, SVN branches appear as remote refs that must be explicitly converted to local branches and tags using the conversion commands in Section 3 of this guide.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| GitHub Enterprise Importer (GEI) & Actions Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Organization Design Patterns (Flat Structure, Teams, Naming) | `Setup/Organization Design Patterns (Flat Structure, Teams, Naming).md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |

---

## 📚 Resources

- [Importing an External Git Repository Using the Command Line](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/importing-an-external-git-repository-using-the-command-line)
- [svn2git (nirvdrum)](https://github.com/nirvdrum/svn2git)

---

*Last updated: April 2026*
