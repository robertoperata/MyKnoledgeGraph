---
tags:
  - persistence
  - hibernate
feature: 
type: course
author: "[[Vlad Mihalcea]]"
source: 
---
# High-Performance Java Persistence

>[!question] Why we care so much about data access performance?
>more than half of application performance bottlenecks originate in the database. The database runs fast as the data access layer allows.

>[!question] what are response time and throughput

>[!question] which are the data access technology stack?

>[!question] JDBC?

>[!question] Jpa and Hibernate?

>[!question] Portability is a feature


![[Screenshot 2025-09-22 at 11.37.37.png]]

The lower the response time, the more responsive an application becomes. Response time is, therefore, the measure of performance. Scaling is about maintaining low response times while increasing system load, so throughput is the measure of scalability.

To measure performance we have **response time** and **throughput**:

$$\begin{aligned}
&\text{Response Time R = }t_{acq} + t_{req}+t_{exec}+t_{res}+t_{idle}
\end{aligned}$$
$$
\begin{aligned}
&\text{n - number of completed transactions}\\[0.5em]
&\text{t - time interval}\\[1em]
&\text{Throughput }X=\frac{n}{t} = \frac{100}{1s}=100 TPS\\[0.5em]
&\text{Average response time }T_{avg}=\frac{t}{n}=\frac{1s}{100}=10ms\\[0.5em]
&X = \frac{1}{T_{avg}}\\
\end{aligned}
$$

USL (Universal Scalability Law) can approximate the maximum relative throughput (system capacity) in relation to the number of load generators (database connections).
$$
\begin{aligned}
&C (N ) = \frac{N}{1 + α (N− 1) + βN (N− 1)}
\end{aligned}
$$
-  $C$ - the ***relative throughput*** gain for the given concurrency level
- $\alpha$ - the ***contention coefficient*** (overhead or serialisable portion of a data processing routine that limits a system's ability to scale with increased concurrency). Contention has the effect of **flattening scalability**, meaning that as more concurrent sessions are added, the throughput gain does not increase proportionally due to resource conflicts. Source of contention are CPUs, memory, disk, locks. When all database server resources are fully utilized, adding more workload to the system will only increase contention, which, in turn, leads to lower throughput
- $\beta$ - the ***coherency coefficient*** (the overhead of maintaining consistency across all concurrent database sessions). High coherency costs can undermine scalability

The number of load generators (database connections), for which the system hits its maximum capacity, depends on the USL coefficients solely.
$$
N_{max} = \sqrt{\frac{(1 - \alpha)}{\beta}}
$$
The resulting capacity gain is relative to the minimum throughput, so the absolute system capacity is obtained as follows:

$$
X_{max} = X(1)\times C(N_{max})
$$
