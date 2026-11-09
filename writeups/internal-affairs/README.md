# Internal Affairs



[Original report, page 8](../../cruXipher_CTF_BreakingBad_Report.pdf#page=8)

**Category** Web

The employee portal at https://internal-affairs.cruxipher.crux-bphc.com asked why Crooxus Dynamics nearly went
bankrupt in February. Inspection of the React bundle, /assets/index-CtrHTARE.js, led to GraphQL at POST /graphql.
The initial session was a manager account, so access to sensitive user fields was restricted.

Schema introspection exposed User.apiKey, User.secretToken, compensation fields and AuditEvent records. The
description of Query.auditEvents instructed clients to send X-API-Key for full access. More importantly, UserFilter
accepted the sensitive credentials even though the corresponding output fields were restricted. Matching user results
therefore acted as a blind prefix oracle.

Aliased batch queries in oracle2.py recovered an administrator's credentials one character at a time. The script is
missing from the archive, so the exact extraction queries cannot be reproduced here. With the recovered API key, the
following query exposed the audit trail:

```
{ auditEvents(limit: 1000) {
    id actor { username } targetUser { username }
    action previousSalary newSalary comment
} }
```

With the recovered API key in the request header, event AUD-2431 showed a self-approved compensation change by
theo.james at 2026-02-18T16:42:09Z: salary increased from 105125 to 42000000. The comment contained:

```
cruXipher{the_rats_read_the_audit_logs}
```

The visible compensation trend supplies a useful consistency check: (765919.921875 - 111312.5) * 64 = 41894875,
exactly the salary increase. The failure was authorization on field output without equivalent protection on filtering,
followed by API-key access to the audit feed. The flag was captured but not submitted.

Investigation sequence

1. Find the API behind the employee portal
The front end was a React application, so its bundled JavaScript was a useful starting point for identifying API paths. The
recorded search against the locally saved bundle was:

```
grep -oE '"/[a-zA-Z0-9_./?=&-]*"' app.js
```

The /graphql string led to schema introspection. This query enumerated type and field names:

```
{ __schema { types { name fields { name } } } }
```

The important distinction was between fields visible in the schema and fields readable under the current session. apiKey
and secretToken were restricted outputs, but their presence as filter inputs created a separate route to learning their
values.

2. Locate the salary anomaly using permitted data
Before escalating access, the compensation trend narrowed the investigation:

```
query {
  compensationTrend {
    date personalSalary averageSalary markedAdjustment
  }
}
```

The marked jump on 18 February pointed to one compensation update rather than a gradual change across the
workforce. The audit event identifier and timestamp then provided a specific target for the privileged query.

3. Use matching results as a credential oracle
The preserved extraction extends a candidate prefix one character at a time and asks whether a user matches it. A
matching account reveals that the candidate is correct even when the credential itself cannot be selected. GraphQL
aliases allow multiple candidate tests in one request. The original oracle2.py is absent, so the precise filter operators
and alias construction are not reproduced as executable queries.

4. Read the full audit event
The recovered administrator API key was sent in X-API-Key, enabling the full auditEvents result. Inspecting actor and
target together established that the same account changed its own compensation. Comparing old and new salary values
gave an increase of 41894875, and the company-average calculation independently matched it. The event comment
supplied the flag; it was not inferred from the salary numbers.

**Source files** combined-writings.md.
