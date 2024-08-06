# MChallenge

## Table of Contents
1. **[Diagnostic Review of Provided Data](#diagnostic-review-of-provided-data)**
2. **[Key Issues Identified](#key-issues-identified)**
3. **[Critical Information Gaps](#critical-information-gaps)**
4. **[Root Cause Analysis](#root-cause-analysis)**
5. **[Proposed Solutions and Workarounds](#proposed-solutions-and-workarounds)**

## Diagnostic Review of Provided Data
This section meticulously examines the data and charts provided by the client to pinpoint the specific performance issues within their Metabase environment.

#### AWS Instance Size and System Load
- **Instance Adequacy**: The current AWS instance size appears to be the minimum required to properly host Metabase, potentially limiting performance during peak loads.
- **Red Flags**: No immediate red flags in the instance specifications, but the minimal provisioning suggests potential vulnerability to spikes in demand.

#### CPU Usage Analysis
- **Daily Peak Patterns**: Notable daily CPU usage peaks at the start of the workday indicate a simultaneous surge in user connections and possibly scheduled alerts, putting strain on resources.
- **Weekend Activity**: Lack of significant activity on 07/16 might suggests lower weekend usage. The system reaches full capacity (100% CPU usage) on 07/17, which could indicate exceptional demand or insufficient scaling.
- **Presto Database Concerns**: Utilisation of Presto for application data is observed, which is contrary to best practices and might introduce inefficiencies or stability issues.

#### Disk I/O Operations
- **Read IOPS Observations**: Peaks in Read IOPS around 9:30 AM support the theory of high initial load as users connect to Metabase. A drop in activity around lunchtime further suggests user behavior significantly impacts system performance.

#### Query Performance
- **Average Active Sessions (AAS)**: An AAS greater than 1 frequently observed, indicating saturation of the database and insufficient capacity to handle concurrent requests smoothly.
- **Bottleneck Identification**: Queries related to information retrieval are causing significant delays, showing high latency, suggesting inefficient query design or indexing.

## Key Issues Identified
Here, we identify and detail the primary technical issues that contribute to the current performance problems experienced by the Metabase instance.

#### Connection Session Overload
- High load (11.56 AAS) and call rate (2.52 calls/sec) for session management query
- Likely overwhelms connection pool and database resources
- High AAS indicates database saturation

#### Database Performance Issues
- CPU Utilization:
    - Daily peaks approaching 100% at workday start
- I/O Bottlenecks:
    - Significant Read IOPS spike around 9:30 AM
- Problematic Queries:
    - High-load UPDATE queries
    - Slow INSERT operations
    - Resource-intensive aggregation queries

#### Architectural Concerns
- Presto Usage:
    - Possibly used for Metabase's application data (not recommended)
    - Potential source of unnecessary complexity and latency

## Critical Information Gaps
This section highlights the crucial data points and system details that are missing, which are necessary to fully diagnose and resolve the performance issues.
- **Details of Deployment Architecture**: Understanding the full architecture setup is vital to assessing whether the current infrastructure supports the operational demands.
- **Database Roles Clarification**: It's unclear if the application database also serves analytical purposes, a distinction crucial for optimizing performance and storage.
- **Scale of Company and User Base**: Knowledge of the company size and active user count is essential for accurately scaling resources and anticipating load challenges.
- **Home Page Layout Complexity**: The complexity and content of the default Metabase home page can greatly influence load times and user experience.
- **Implementation of Caching Mechanisms**: Information on whether caching is implemented, and how it's managed, is critical for understanding data retrieval efficiencies.
- **Configuration of Alerts**: The specifics of how alerts are configured and scheduled can significantly impact system performance during peak operational hours.

## Root Cause Analysis
We analyze the underlying factors contributing to the identified issues, providing a clearer picture of what's causing the system's inefficiencies.
- **Inefficient Session Management**: Excessive concurrent sessions strain database connections and resources, causing bottlenecks.
- **Database Performance Problems**: The database struggles with high CPU and I/O loads during peak times.
- **Possible Misuse of Presto**: Presto is inappropriately used for application data, adding complexity and slowing responses.
- **Under-provisioned Infrastructure for Peak Loads**: Infrastructure isn't scaled to handle peak traffic, leading to performance issues.
- **Potentially Resource-Intensive Queries Triggered on Home Page Access**: Automatic heavy queries on home page access drain resources during peak logins.

## Proposed Solutions and Workarounds
This section proposes immediate and medium-term actions to alleviate the identified issues and enhance overall system performance.

### Immediate Actions:
- **Restart the Instance**
    - **Action**: Provides temporary relief by clearing current session and connection buildup.
    - **Justification**: Effective for immediately resolving memory leaks or clearing stuck processes, as recommended in Metabase’s operational best practices.
    - **Potential Risks**: The restart may disrupt user activity, and frequent restarts could indicate underlying unresolved issues that need more permanent solutions.
    - **Impact**: This action can quickly alleviate system strain and improve response times, albeit temporarily.
- **Increase the Instance Server Size**
    - **Action**: Expands server capacity to better handle current and near-future workloads.
    - **Justification**: Supported by historical data showing CPU usage peaks at 100% during workday starts, indicating insufficient capacity for peak loads.
    - **Potential Risks**: Higher operational costs and the possibility of over-provisioning, which could lead to unnecessary expenditure if not regularly evaluated.
    - **Impact**: A larger server can manage increased user demand and reduce the risk of performance bottlenecks, enhancing stability and user experience.
- **Remove Specific Dashboards from Metabase Home**
    - **Action**: Simplifies the initial load by reducing the data processed on the home page.
    - **Justification**: Dashboard complexity is a known cause of slow load times; reducing complexity can significantly improve initial access times.
    - **Potential Risks**: Users may lose quick access to key data previously available at login, potentially impacting their workflow or satisfaction.
    - **Impact**: Users experience faster access and navigation, improving overall user satisfaction and system efficiency.

### Medium term actions:
- **Implement Time-Based Horizontal Scaling**
    - **Action**: Dynamically adjust the number of Metabase instances to align with fluctuations in daily traffic.
    - **Justification**: Matches computing resources with user demand, preventing both underutilization and overload.
    - **Potential Risks**: Complexity in implementation, potential for misconfiguration, and increased management overhead.
    - **Impact**: Ensures optimal resource allocation, maintaining performance during peak periods without over-provisioning during low-usage times.
- **Implement Query Caching**
    - **Action**: Cache results of frequently run queries to reduce redundant processing.
    - **Justification**: Improves speed and efficiency by preventing repeated execution of the same queries.
    - **Potential Risks**: Stale data issues if not properly invalidated; increased memory usage.
    - **Impact**: Decreases direct database access for common queries, significantly speeding up response times and reducing load.
- **Review and Adjust Connection Pooling Settings**
    - **Action**: Optimize how database connections are managed and allocated across user requests.
    - **Justification**: Ensures that database connections are efficiently used and not a bottleneck.
    - **Potential Risks**: Improper settings can lead to resource exhaustion or underutilization.
    - **Impact**: Improves database efficiency and stability, ensuring smoother user experiences under varying load conditions.
- **Evaluate Presto Usage**
    - **Action**: Assess the current use of Presto and consider transitioning to PostgreSQL for application data management.
    - **Justification**: Targets optimal use of technology tailored to application needs and performance characteristics.
    - **Potential Risks**: Transition risks, including downtime and data migration challenges; compatibility issues.
    - **Impact**: Potentially simplifies the technology stack, reducing overhead and improving performance and scalability.
- **Monitor Performance with Metabase Analytics (Pro/Enterprise)**
    - **Action**: Use advanced analytics tools to continuously track and analyze performance metrics and user activity.
    - **Justification**: Enables data-driven decisions for tuning and scaling.
    - **Potential Risks**: May require significant resources for setup and ongoing management; privacy concerns if not properly managed.
    - **Impact**: Enables proactive identification and resolution of performance bottlenecks, enhancing overall system efficiency.
