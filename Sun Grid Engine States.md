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

<table>
<thead>
<tr>
<th>Category</th>
<th>State</th>
<th>SGE Letter Code</th>
</tr>    
</thead>
<tr>
<td valign="top" rowspan="7">Pending</td>
<td>pending</td>
<td>qw</td>
</tr>
<tr>
<td>pending, user hold</td>
<td>qw</td>
</tr>
<tr>
<td>pending, system hold</td>
<td>hqw</td>
</tr>
<tr>
<td>pending, user and system hold</td>
<td>hqw</td>
</tr>
<tr>
<td>pending, user hold, re-queue</td>
<td>hRwq</td>
</tr>
<tr>
<td>pending, system hold, re-queue</td>
<td>hRwq</td>
</tr>
<tr>
<td>pending, user and system hold, re-queue</td>
<td>hRwq</td>
</tr>
<tr>
<td valign="top" rowspan="4">Running</td>
<td>running</td>
<td>r</td>
</tr>
<tr>
<td>transferring</td>
<td>t</td>
</tr>
<tr>
<td>running, re-submit</td>
<td>Rr</td>
</tr>
<tr>
<td>transferring, re-submit</td>
<td>Rt</td>
</tr>
<tr>
<td valign="top" rowspan="4">Suspended</td>
<td>job suspended</td>
<td>s, ts</td>
</tr>
<tr>
<td>queue suspended</td>
<td>S, tS</td>
</tr>
<tr>
<td>queue suspended by alarm</td>
<td>T, tT</td>
</tr>
<tr>
<td>all suspended with re-submit</td>
<td>Rs, Rts, RS, RtS, RT, RtT</td>
</tr>
<tr>
<td>Error</td>
<td>all pending states with error</td>
<td>Eqw, Ehqw, EhRqw</td>
</tr>
<tr>
<td>Deleted</td>
<td>all running and suspended states with deletion</td>
<td>dr, dt, dRr, dRt, ds, dS, dT, dRs, dRS, dRT</td>
</tr>
</table>

Sources
-------

-   [SGE 6 Usage for
    Users (NASA)](http://ceres.larc.nasa.gov/documents/presentations/05-SGE-6-Usage-For-Users.pdf)
