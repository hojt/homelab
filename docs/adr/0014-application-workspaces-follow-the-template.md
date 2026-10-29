# ADR-0014: Application Workspaces Follow the Template

**Status:** Accepted

**Refines:** [ADR-0007](0007-application-workspace-structure.md), [ADR-0009](0009-application-workspace-composition.md)

## Context

Application workspaces are created by copying `template-app-workspace`.
After the copy, an application has no connection to the template. Improvements
to the template, such as changes to the coding-agent sandbox, the developer
environment, or agent instructions, do not reach applications created earlier.
Each application must be updated by hand, or not at all.

`example-app` and the template share most of their workspace infrastructure,
but are separate repositories without a common history. Every change to that
infrastructure has been made twice, and the two have drifted apart, most
visibly in their documentation.

The number of application workspaces is expected to grow. The cost of keeping
them aligned by hand grows with it.

Git can carry changes between repositories that share history. An application
that keeps the template as an upstream repository can merge later template
versions, and Git shows exactly where the application has diverged from the
template.

## Decision

Application workspaces keep the template as an upstream repository and receive
its improvements by merging released template versions.

### The template is released in versions

The template marks released versions with tags. Applications merge tagged
versions, not arbitrary template commits.

Everything on the template's default branch reaches every application that
merges it. Experiments with the template happen on branches or in separate
copies, not on its default branch.

### Ownership of files is explicit

The template declares which files it owns. Template-owned files are the same
in every application workspace. Applications do not change them; a change to a
template-owned file is made in the template and reaches applications through a
template version.

Other files in the template are starting points. They belong to the
application once it has been created, and the application changes them freely.
The template changes starting points only when necessary, since such changes
conflict with applications that have changed them.

Files that mix template-owned and application-owned content are split so that
each file has a single owner.

### Updates are explicit and reviewed

The developer decides when an application merges a template version. The merge
is reviewed and committed like any other change.

Template updates are recorded as merge commits. They are an exception to a
preference for linear history: merging keeps the connection to the template
that later updates rely on. Template updates are never squashed, and
applications are never rebased onto the template.

### Removed template content stays removed

Applications replace the template's example components with their own. When a
template version changes content that an application has removed, the removal
is kept.

### `example-app` is an application created from the template

`example-app` keeps the template as its upstream and receives template versions
like any other application. It remains the reference application that runs on
the platform, and thereby the application that shows that a template version
works.

`example-app` and the template are connected once, by a merge of their
unrelated histories. Afterwards, template-owned files are changed in the
template only.

## Consequences

Improvements to the template reach every application that merges a new
version, including applications created before the improvement.

Workspace infrastructure is changed in one place. Changes are developed and
tested in the template, and validated by merging them into `example-app`,
reversing the current practice of changing `example-app` first.

Each template update is a merge commit in the application's history, so the
history shows when each application adopted which template version.

Changes to starting points, and to content that an application has removed,
produce conflicts. Conflicts on removed content are resolved by keeping the
removal and can be resolved mechanically. Conflicts on starting points show
where the application has diverged and are resolved by the developer.

The template's default branch must be kept free of experiments and unfinished
work, since anything merged there spreads to every application that merges a
later version.

An application that needs to change a template-owned file must either change
the template or accept that the change is overwritten by the next template
merge.

Connecting `example-app` and the template requires one merge with conflicts
across most shared files. Applications created from the template after this
decision start with the template's history and need no such step.

Existing application workspaces adopt this decision through separate,
validated changes. This decision does not claim that any workspace already
follows it.
