# Coaches dashboard: one repo to create issues that land on the right team's board

This is the other direction from the team-repo setup you already have. You
open an issue here, pick a team from a dropdown, and it gets copied into
that team's repo - which already has `add-to-project.yml` watching for new
issues, so from there it flows onto the org board automatically, tagged
with that team, with no extra work.

## 1. Create the dashboard repo

github.com/Avenues-Robotics -> **New repository** -> name it something
like `coaches-dashboard` -> Create. It doesn't need any special settings;
Issues are on by default, which is all it uses.

## 2. Add it to the PROJECT_PAT secret's access list

This repo needs to read the same `PROJECT_PAT` secret your team repos
already use, since creating an issue in another repo needs that token, not
the default one Actions gives a workflow.

Org page -> Settings -> Secrets and variables -> Actions -> `PROJECT_PAT`
-> add `coaches-dashboard` to its selected repositories.

## 3. Add the project fields these new form fields need

Your project already has a built-in **Status** field (every new project
gets one) - open it and check its options are exactly `Todo`, `In
Progress`, `Done`. Rename or reorder if needed; you don't need to recreate
it.

Then add three more fields the same way you added **Team** originally
(the project's ... menu -> Settings -> Fields -> **New field**):

- **Type** - Single select - options: `Task`, `Bug`, `Feature`
- **Priority** - Single select - options: `Must`, `Should`, `Could`,
  `Won't (this time)`
- **Start date** - Date
- **Target date** - Date

You can skip any of these you don't care about tracking on the board - the
relay workflow just skips setting a field that doesn't exist on the
project (with a note in its log), it won't fail because of it.

## 4. Add the issue template

In `coaches-dashboard`, create a file at exactly this path (when using
"Add file" -> "Create new file" on github.com, you can type the whole path
including the folders into the filename box and it creates them for you):

```
.github/ISSUE_TEMPLATE/team-task.yml
```

Paste in `team-task.yml`'s contents. Before committing, check its `options:`
list under `Team` - it only has `Tritonics` right now. Add one line per
team you coach, exactly matching the team names you're already using in
each repo's `add-to-project.yml`. The Status/Type/Priority options in this
file already match step 3 above - only touch them if you changed the
project's own option names.

There's no assignee field in this template - that's intentional, see the
note at the top of the file.

## 5. Add the relay workflow

Same idea, a new file at:

```
.github/workflows/relay-to-team-repo.yml
```

Paste in `relay-to-team-repo.yml`'s contents. Before committing, check two
things near the top:

- `PROJECT_NUMBER` - set to your org project's number (same one used in
  every repo's `add-to-project.yml`). It's `"9"` here, already matching
  what you've been using.
- `TEAM_REPO_MAP` - only maps `Tritonics` to `17253_BIOBUZZ` right now.
  Add one line per team, mapping that team's exact name (matching what you
  put in step 4) to their repo's name:

```yaml
TEAM_REPO_MAP: |
  {
    "Tritonics": "17253_BIOBUZZ",
    "Iron Eagles": "some-other-repo-name"
  }
```

## 6. Try it

`coaches-dashboard` -> Issues -> New issue -> pick the "Team task" template
-> choose a team, fill in the fields, optionally pick an assignee in the
sidebar -> Submit.

Within a few seconds: a new issue appears in that team's repo, assigned to
whoever you picked (which their `add-to-project.yml` picks up and adds to
the org board), its Status/Type/Priority/dates get set on the board item,
and the original issue here gets a comment linking to it and closes
itself. Check the Actions tab on `coaches-dashboard` if something's
missing - the run's log will say exactly what happened, including which
fields it skipped and why.

## Keeping it in sync

Three places have to agree on a team's exact name: the `Team` dropdown
(step 4), `TEAM_REPO_MAP` (step 5), and that team's `TEAM_NAME` in their
own repo's `add-to-project.yml`. Adding a new team means touching all
three. A mismatch anywhere fails loudly in the Actions log rather than
silently misrouting a task, so it's easy to spot, just easy to forget a
step - keeping a short checklist of your teams and their repo names
somewhere handy is worth it once you have more than two or three.

Likewise, if you ever rename an option on the project's Status, Type, or
Priority field, update the matching `options:` list in `team-task.yml` to
match - otherwise new issues will still offer the old name, and the relay
workflow won't find a matching option to set on the board.

## Note on Status, Type, Priority and the dates

These only get set on the board item for issues created through this
dashboard. An issue opened directly in a team's own repo (bypassing the
dashboard) still gets added to the board and tagged with its team as
before, just without these extra fields - there's nowhere for a student to
enter them from that side. If you want that changed later, it would mean
teaching each repo's own `add-to-project.yml` the same field-parsing this
one does, which is a bigger change since it touches every team repo.
