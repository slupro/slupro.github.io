# Designing Data Intensive Applications (DDIA)

In DDIA, it focus on 3 concerns:

> 1. Reliability: Continuing to work correctly, even when things go wrong.
> 2. Scalability: Scale up, or scale out.
> 3. Maintainability: Operability, Simplicity, Extensibility.


The architecture of systems that operate at large scale is usually highly specific to the
application—there is no such thing as a generic, one-size-fits-all scalable architecture
(informally known as magic scaling sauce). The problem may be *the volume of reads*,
*the volume of writes*, *the volume of data to store*, *the complexity of the data*, *the
response time requirements*, *the access patterns*, or (usually) some mixture of all of
these plus many more issues.



> Fault vs Failure

A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user.

