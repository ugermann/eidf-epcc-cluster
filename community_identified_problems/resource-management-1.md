## Better Cluster Resource Utilization - Priority 1

The cluster's resource utilization needs immediate improvement. Here's what needs to be addressed:

### Node Usage Bands and Automated Pod Termination

#### Goals:

- Eliminate tricks like "while true; do sleep 60; done," which keeps jobs running indefinitely.
- Stop low GPU utilization and resource hoarding.
- Curtail excessive job durations that sidestep priority scheduling.

#### Proposals:

1. **Job Length Bands**: Implement tiers for job durations (e.g., 12, 72 hours, and 7 days). Limit the number of nodes each user can use in each tier.

2. **Usage Reports**: Supply users with daily or weekly GPU utilization reports to encourage self-regulation.

### Exceptions and Whitelist Policy

#### Goals:

- Establish clear guidelines for jobs requiring extended runtimes.

#### Proposals:

1. **Moderated Extensions**: Appoint moderators to review and approve extended run time requests. All requests and approvals should be public, allowing for community input and appeals.

Let's act now to make the cluster more efficient and equitable for all users.