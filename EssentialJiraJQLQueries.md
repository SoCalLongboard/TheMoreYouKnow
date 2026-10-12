# Essential Jira JQL Queries

A starter set of saved filters for knowing what work is ahead of you, in the current sprint or in general.

> Replace `PROJ` with your project key where needed, or drop the project clause if you only have one board. Status names (e.g. `"In Review"`) vary by workflow, so adjust them to match yours.

## Contents

- [Your Current Sprint](#your-current-sprint)
- [Beyond the Sprint](#beyond-the-sprint)
- [Awareness and Hygiene](#awareness-and-hygiene)
- [Testing-Focused Extras](#testing-focused-extras)
- [Tips](#tips)

---

## Your Current Sprint

### 1. My open work this sprint

```sql
assignee = currentUser() AND sprint in openSprints() AND statusCategory != Done ORDER BY priority DESC, rank ASC
```

### 2. My in-progress items

Good for standup.

```sql
assignee = currentUser() AND sprint in openSprints() AND statusCategory = "In Progress"
```

### 3. Blocked or flagged, mine

```sql
assignee = currentUser() AND sprint in openSprints() AND (Flagged is not EMPTY OR status = Blocked)
```

### 4. What I finished this sprint

Handy for demos and retros.

```sql
assignee = currentUser() AND sprint in openSprints() AND statusCategory = Done
```

---

## Beyond the Sprint

### 5. Everything assigned to me, not done

```sql
assignee = currentUser() AND statusCategory != Done ORDER BY updated DESC
```

### 6. Assigned to me but not in any sprint

Stuff that's quietly accumulating.

```sql
assignee = currentUser() AND sprint is EMPTY AND statusCategory != Done
```

### 7. Next sprint, mine

```sql
assignee = currentUser() AND sprint in futureSprints()
```

### 8. Unassigned in the current sprint

Work nobody has claimed.

```sql
sprint in openSprints() AND assignee is EMPTY AND statusCategory != Done
```

---

## Awareness and Hygiene

### 9. Waiting on me for review/testing

```sql
status in ("In Review", "Ready for QA", "Code Review") AND assignee = currentUser()
```

### 10. Things I reported that are still open

Follow-ups you're waiting on others for.

```sql
reporter = currentUser() AND assignee != currentUser() AND statusCategory != Done
```

### 11. Recently updated items I'm involved in

```sql
(watcher = currentUser() OR assignee was currentUser() OR commentedBy = currentUser()) AND updated >= -3d ORDER BY updated DESC
```

### 12. Stale items of mine

Nothing touched in two weeks.

```sql
assignee = currentUser() AND statusCategory != Done AND updated <= -14d
```

### 13. Sprint scope creep check

```sql
sprint in openSprints() AND created >= startOfWeek() AND issuetype != Epic
```

For items added *after* the sprint started, the built-in sprint report tracks that natively.

---

## Testing-Focused Extras

### Open bugs I've filed

```sql
reporter = currentUser() AND issuetype = Bug AND statusCategory != Done
```

### Ready for QA with nobody assigned to verify

```sql
status = "Ready for QA" AND assignee is EMPTY
```

---

## Tips

- **Prefer `statusCategory` over specific status names.** It survives workflow customization and works across projects with different status names.
- **Save each query as a filter and star it**, then add the key ones to your Jira dashboard as filter-results gadgets.
- **`ORDER BY Rank`** matches the board's priority order, which is what the team has agreed on, so it's usually a better default than sorting by Priority alone.
- **Subscribe to #1 or #12** as a filter subscription (a daily morning email) for a nudge without opening Jira.
- **Kanban or no sprints?** Drop the `sprint` clauses and lean on #5, #9, and #12.
- Jira Cloud's **"Your work"** page covers some of this out of the box, but JQL filters give you more control and are shareable with teammates.
