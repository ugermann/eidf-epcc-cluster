# All-Hands Meeting on Cluster Utilization: Detailed Topics for Discussion

## 1. **Resource Utilization Rules**
   
### Subtopic A: Maximum GPU Utilization Time
   - **Nuance:** Ending the "while true; do sleep 60; done;" trick; Different bands for different utilization periods, such as 2-12 hours, 12-24 hours, 1-3 days, etc.
   - **Existing Ideas:** Each band may have its own set of rules or moderation.

### Subtopic B: Multi GPU Node Utilization
   - **Nuance:** Utilization for debugging purposes.
   - **Existing Ideas:** Allowing 1 multi-GPU node for up to 8 hours with a "_dev" identifier.

## 2. **Automated Termination Policy**
   
### Subtopic A: Termination of Inactive Jobs
   - **Nuance:** Consideration for idle periods.
   - **Existing Ideas:** Checking GPU utilization with `nvidia-smi` and killing jobs not hitting a threshold.

### Subtopic B: Exceptions and Whitelist
   - **Nuance:** Agreement on job names that need extended run times.
   - **Existing Ideas:** 2-3 moderators to accept or reject requests for longer run times.

## 3. **Written Policies and Procedures**
   - **Nuance:** Establishing a clear written policy to guide users.
   - **Existing Ideas:** Temporary procedure using automation scripts, and eventual development of more long-term solutions.

## 4. **Kubernetes vs. SLURM Configuration**
   - **Nuance:** Kubernetes capabilities in job scheduling.
   - **Existing Ideas:** Circumventing the "while true" trick with various Kubernetes configurations.

## 5. **Moderation and Community Interaction**
   - **Nuance:** Community engagement in policy decision-making.
   - **Existing Ideas:** Voting system, slack interaction, and collaborative whitelist management.

## Voting Procedure
   - **In-Meeting Voting:** A voting system will be used during the meeting to gather opinions on each topic and subtopic.
   - **Post-Meeting Voting:** Those not present during the meeting will have a short window to voice their opinions on Slack or GitHub.

## Next Steps
Following the meeting, the decisions reached will be documented and shared with the entire community. Implementation will proceed in accordance with the agreed-upon policies and procedures.

---

[Please submit any additional topics or ideas to be considered prior to the meeting.](#link-to-form)
