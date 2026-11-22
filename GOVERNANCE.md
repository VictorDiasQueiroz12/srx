# Maintainer governance and continuity

## Maintainer responsibilities

`@fullzer4` is the current write maintainer and owns issue and PR triage,
contribution decisions, release coordination, repository access review and
recovery. The maintainer also handles private reports under the
[security policy](SECURITY.md) and coordinates the
[release and recovery procedure](docs/security/releases.md).

## Contribution decisions and escalation

Anyone can propose changes and give feedback in GitHub issues and PRs. The
maintainer decides whether to accept, request revisions, defer or decline a
proposal based on project scope, correctness, security, compatibility and
maintenance cost. Record the decision and rationale in the relevant issue or PR,
including what would allow a deferred or declined proposal to be reconsidered
when applicable. Keep sensitive details in the private reporting channel.

Ask for reconsideration in the original issue or PR with new evidence or an
alternative. If a conflict of interest or maintainer unavailability prevents a
decision, pause the affected work and ask a `ryro-hq` organization owner to
coordinate resolution. Do not use public issues for vulnerability details,
credentials or recovery evidence; follow [SECURITY.md](SECURITY.md) for private
reports and its fallback contact.

All changes still follow [CONTRIBUTING.md](CONTRIBUTING.md): each author certifies
their own contribution under the DCO, required `verify` and `DCO` checks must
pass, conversations must be resolved, and integration uses the required rebase
merge queue. Neither a dispute nor a recovery authorizes a bypass.

## Independent reviewer

A candidate must be a distinct human using their own GitHub account with MFA,
with a record of substantive contributions or reviews and familiarity with the
project's testing, security and contribution policies. They must be willing to
withhold approval and available to review on an ongoing basis. For a specific
PR, the reviewer must not have authored or co-authored the change and must
recuse themselves if a conflict could compromise their judgment. Self-review,
alternate accounts and automation do not count as independent review.

1. Start with public shadow reviews without elevated access. The maintainer
   evaluates the candidate's feedback and confirms that they can assess the
   change, test evidence and security impact independently. Shadow reviews do
   not close the documented gap.
2. Before any access change, document the candidate's consent, qualifications,
   review scope and permissions needed, and follow the access review below.
   Reviewer status does not imply administrative, release or recovery access.
3. After authorized onboarding, verify effective access and record a
   substantive GitHub review on a real PR, with approval only when warranted.
   Making independent approval mandatory requires a separately authorized
   protection change and verification that it is enforced.

There is currently **one write maintainer and no independent human reviewer
available**. Zero required approvals is temporary and is not evidence of
independent review. Keep the gap open in the
[controls record](docs/security/controls.md) until a real reviewer is onboarded
and review evidence exists. Reviewer availability alone does not satisfy
OSPS-QA-07.01; that control also requires enforced non-author human approval.
If reviewer coverage is lost, document the gap again without weakening
protections.

## Access review, succession and recovery

Before any grant of elevated access, the maintainer reviews the candidate,
confirms MFA readiness and documents the duties, least-privilege scope and
decision. An authorized administrator applies changes only after separate
authorization. The maintainer also reviews effective access quarterly and after
role changes, checking organization and repository roles, team and direct
grants, Apps and release access against current duties. Record the review date,
owner, findings and follow-ups without publishing sensitive access details.
Handle unnecessary access or suspected compromise through authorized
administrators and the [recovery procedure](docs/security/releases.md).

For a planned handoff, the current maintainer proposes a willing successor who
meets the reviewer readiness criteria and can take on triage, security reporting,
access review and recovery. Document who owns each duty, outstanding work and
how to reach the escalation contact. An authorized organization owner confirms
the handoff and separately manages any necessary access changes; review effective
access and update the documented contacts once responsibilities transfer. Use
individual accounts and never hand over personal credentials or recovery codes.

If the maintainer is unavailable or compromised, an authorized organization
owner coordinates recovery and confirms who will take responsibility for it.
Until authorized ownership is restored, pause merges and releases that cannot
proceed through the normal controls. The maintainer owns periodic recovery
exercises and records the outcome and follow-ups. The
[v0.1.0 qualification record](docs/security/release-qualification-v0.1.0.md#installation-example-and-recovery)
records a local quarantine drill; a non-production credential revocation and
replacement exercise remains pending. This policy documents a process; it does
not grant access, appoint a successor or change protections.
