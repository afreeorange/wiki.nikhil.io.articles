Queues
------

-   **u** - Unknown
-   **a** - Alarm (load threshold reached)
-   **A** - Suspend threshold reached
-   **s** - Suspended by user or admin
-   **d** - Disabled by user or admin
-   **C** - Suspended by calendar
-   **D** - Disabled by calendar
-   **S** - Suspended by subordination
-   **E** - Error! `sge_execd` can't reach shepherd

Jobs
----

### States

-   **q** *ueued*
-   **d** *eletion*
-   **t** *ransfering*
-   **r** *unning*
-   **R** *estarted*
-   **s** *uspended*
-   **S** *uspended*
-   **T** *hreshold*
-   **w** *aiting*
-   **h** *old*

### Table

<table markdown="1" class="wikitable" cellpadding="4">
<tr markdown="1">
<th markdown="1">
Category

</th>
<th markdown="1">
State

</th>
<th markdown="1">
SGE Letter Code

</th>
</tr>
<tr markdown="1">
<td markdown="1" valign="top" rowspan="7">
Pending

</td>
<td markdown="1">
pending

</td>
<td markdown="1">
qw

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, user hold

</td>
<td markdown="1">
qw

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, system hold

</td>
<td markdown="1">
hqw

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, user and system hold

</td>
<td markdown="1">
hqw

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, user hold, re-queue

</td>
<td markdown="1">
hRwq

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, system hold, re-queue

</td>
<td markdown="1">
hRwq

</td>
</tr>
<tr markdown="1">
<td markdown="1">
pending, user and system hold, re-queue

</td>
<td markdown="1">
hRwq

</td>
</tr>
<tr markdown="1">
<td markdown="1" valign="top" rowspan="4">
Running

</td>
<td markdown="1">
running

</td>
<td markdown="1">
r

</td>
</tr>
<tr markdown="1">
<td markdown="1">
transferring

</td>
<td markdown="1">
t

</td>
</tr>
<tr markdown="1">
<td markdown="1">
running, re-submit

</td>
<td markdown="1">
Rr

</td>
</tr>
<tr markdown="1">
<td markdown="1">
transferring, re-submit

</td>
<td markdown="1">
Rt

</td>
</tr>
<tr markdown="1">
<td markdown="1" valign="top" rowspan="4">
Suspended

</td>
<td markdown="1">
job suspended

</td>
<td markdown="1">
s, ts

</td>
</tr>
<tr markdown="1">
<td markdown="1">
queue suspended

</td>
<td markdown="1">
S, tS

</td>
</tr>
<tr markdown="1">
<td markdown="1">
queue suspended by alarm

</td>
<td markdown="1">
T, tT

</td>
</tr>
<tr markdown="1">
<td markdown="1">
all suspended with re-submit

</td>
<td markdown="1">
Rs, Rts, RS, RtS, RT, RtT

</td>
</tr>
<tr markdown="1">
<td markdown="1">
Error

</td>
<td markdown="1">
all pending states with error

</td>
<td markdown="1">
Eqw, Ehqw, EhRqw

</td>
</tr>
<tr markdown="1">
<td markdown="1">
Deleted

</td>
<td markdown="1">
all running and suspended states with deletion

</td>
<td markdown="1">
dr, dt, dRr, dRt, ds, dS, dT, dRs, dRS, dRT

</td>
</tr>
</table>
Sources
-------

-   [SGE 6 Usage for
    Users (NASA)](http://ceres.larc.nasa.gov/documents/presentations/05-SGE-6-Usage-For-Users.pdf)
