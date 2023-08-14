# All-Hands Meeting on Cluster Utilization: Detailed Topics for Discussion

## 1. **Voting Procedure and Community Engagement**
   **Nuance:** Explaining how the voting will work, engaging users through various platforms, and providing an opportunity for post-meeting appeals/thoughts.
   - **Summary:** Invitation will be published through Slack, GitHub, and the mail group. Users missing the meeting will have 7 days to post their appeal/thoughts before new policies are installed. It's an imperfect system but aims to cater to those who use the cluster and engage with the community.
   - **Existing Ideas:**
      - Multichannel engagement to ensure maximum participation.
      - Clear communication on voting process, timelines, and mechanisms for appeals or feedback.
      - Commitment to a transparent and inclusive process that acknowledges limitations but strives for community-driven decision-making.
      - For any policies that relate to the community, the votes will be what decides if a new policy will be installed.
      - For any policies that require the direct agreement of the EIDF folks, the opinions and votes will be presented, and a case will be made. The final say will come directly from the EIDF people.
      - All Hands-on Board Meetings once every 3 months, or, as needed depending on demand.

## 2. **Better Resource Utilization**

### Subtopic A: Node Usage Bands and Automated Pod Termination Policy
   - **Goals:**
      - Ending the "while true; do sleep 60; done;" trick;
      - Preventing hoarding of resources with low GPU utilization.
      - Preventing users from hoarding nodes by having extremely long job durations (this may circumvent any priority-scheduling system hence why it needs its own way of prevention) 
   - **Existing Ideas:**
      - Autoterminating jobs with GPU utilization under a specific amount (i.e. Checking GPU utilization with `nvidia-smi` and killing jobs not hitting a threshold).
      - Having multiple job length bands, each user can only use a certain number of nodes for each band (e.g. 12 hours, 72 hours, 7 days).

### Subtopic B: Development Nodes
   - **Nuance:** Utilization of N-GPU nodes for debugging purposes.
   - **Existing Ideas:** Allowing 1 multi-GPU node for up to 8 hours with a "_dev" identifier.

### Subtopic C: Exceptions and Whitelist
   - **Nuance:** Agreement on job names that need extended run times.
   - **Existing Ideas:** 2-3 moderators to accept or reject requests for longer run times. Requests should be made publicly in relevant Slack channels and/or GitHub page, and publicly accepted with stated reasons. Anyone can appeal or express concerns/comments.

## 3. **Implementation of ReadWriteMany for PVCs**
  - **Nuance:** Transition from ReadWriteOnce to ReadWriteMany: Currently, the cluster supports PVCs with ReadWriteOnce access mode, restricting the volume to be mounted as read/write by a single node only. This limitation can lead to challenges in scenarios where multiple nodes need concurrent read/write access to the same volume, such as collaborative projects, parallel processing tasks, or shared resource handling. The support team is urged to implement the ReadWriteMany access mode for PVCs, which would allow the volume to be mounted by multiple nodes, enhancing:
  Collaboration: Enables different team members to work on shared datasets or projects simultaneously.
  Scalability: Allows parallel processing and distributed computing, enhancing efficiency and reducing processing time.
  Flexibility: Facilitates easier management of shared resources and adaptability to diverse workflows.
  Consistency and Integrity: Ensures that data remains consistent across different nodes, preserving integrity.
  This change would align with the collaborative and dynamic nature of modern workflows, providing a more robust and flexible environment for various operations on the cluster.

## 4. **Kubernetes vs. SLURM Configuration**
   - **Nuance:** Some users consider SLURM a good scheduler to have side by side with Kubernetes. While Kubernetes is the selected solution for this cluster, we wish to hear what people think. Slurm can be implemented as a job on a Kubernetes cluster, so perhaps this is something we could propose if there is significant value to it.
   - **Existing Ideas:** **Open to proposals**.

## 5. **Moderation and Community Interaction**
   - **Nuance:** Community engagement in policy decision-making.
   - **Existing Ideas:**
      - All hands meetings every 3 months.
      - Problem/Solution proposal and Voting system.
      - Slack/Github interaction, and collaborative whitelist management.

## 6. **Cluster Etiquette**
   **Nuance:** Agreement on proper use of login node, worker nodes, persistent storage volumes, etc.
   - **Existing Ideas:** (add existing ideas here)

## 7. **Download Connection Issues: 
   **Summary:** Some links that work from local machines do not work on the cluster, i.e., TensorFlow datasets, Hugging Face uploading breaking all the time while locally it's fine, downloading a dataset from a .ed.ac.uk server location (cocostuff) also failing.
   - **Existing Ideas:** Review of policies and potential technical solutions to address connectivity and download issues. Consideration of disk space requirements for various user needs and exploration of possibilities for extension.

## 8. **Login Node Resource Shortages**:
 **Summary**: The login node has already run out of CPU and disk space in a few separate occasions. A larger disk space and more CPU cores have been added to improve the issues. However, we wanted to keep this issue open for any observations or proposals to further improve the situation.
 **Existing Ideas:**
 - Set up resource monitoring alarms that immediately tell support that CPU, disk, RAM, or network bandwidth are at dangerous level of utilization on the login node, so that someone can have a look quickly to prevent any user interruption.
      
## 9. **How Can One Use the Cluster Without Using the Login Node?**
   **Nuance:** Discussion on alternative methods for accessing and utilizing the cluster without going through the login node.
   - **Existing Ideas:**
      - Exploration of alternative access methods that bypass the login node.
      - Documentation and guidance on implementing these methods.
      - Potential considerations for security, efficiency, and user experience.
        
## 10. **Improving Support Response Time**
   **Nuance:** Addressing concerns about current support being too slow (e.g. 2 weeks or longer for disk expansion on login node)
   - **Summary:** An emphasis on quicker response times and more autonomy for community members in handling technical issues within the cluster.
   - **Existing Ideas:**
      - **Support Availability:** Proposal to have at least one staff member available on Slack to respond within 2 hours of a request or less.
      - **Privilege Elevation:** Discussion about granting higher privilege status to certain community members to quickly resolve issues, while adhering to the necessary security and policy considerations.



## Next Steps
Following the meeting, the decisions reached will be documented and shared with the entire community. The new policies will be proposed to the EIDF staff members and a decision will be made by them, the details of which will be shared with the community. Implementation will proceed in accordance with the agreed-upon policies and
