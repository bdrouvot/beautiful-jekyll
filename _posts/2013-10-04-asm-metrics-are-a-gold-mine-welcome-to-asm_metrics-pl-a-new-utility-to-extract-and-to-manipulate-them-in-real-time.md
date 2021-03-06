---
layout: post
title: ASM metrics are a gold mine. Welcome to asm_metrics.pl, a new utility to extract
  and to manipulate them in real time
date: 2013-10-04 09:51:14.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ASM
- Perl Scripts
- ToolKit
tags: [ASM, oracle]
meta:
  _edit_last: '40807211'
  _publicize_pending: '1'
  _wpas_skip_2225791: '1'
  publicize_twitter_user: BertrandDrouvot
  publicize_twitter_url: http://t.co/9yuWaWmujy
  _wpas_done_2225791: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:246399475;b:1;}}
  publicize_linkedin_url: http://www.linkedin.com/updates?discuss=&scope=16310177&stype=M&topic=5791816566731460608&type=U&a=YpO_
  _wpas_done_2077996: '1'
  _wpas_skip_2077996: '1'
  _oembed_125b05f304f74a2f5ae3c54781c7a8f1: "{{unknown}}"
  _oembed_568e8991a1a026509ec8058ae9ce03f6: "{{unknown}}"
  _oembed_52c508068fafbdd81612121e591ad15d: "{{unknown}}"
  _oembed_5b079673c94d23c4081082ae4ac44f7d: "{{unknown}}"
  _oembed_b17cf7626bbf1e3bb3777a029fc8a1e4: "{{unknown}}"
author:
  login: bdrouvot
  email: bdtoracleblog@gmail.com
  display_name: bdrouvot
  first_name: ''
  last_name: ''
permalink: "/2013/10/04/asm-metrics-are-a-gold-mine-welcome-to-asm_metrics-pl-a-new-utility-to-extract-and-to-manipulate-them-in-real-time/"
---

First of all, I would like to mention that my asmiostat utility has been extracted from my [real\_time.pl](http://bdrouvot.wordpress.com/real_time/ "real_time") script. This is due to the fact that **I just added a new feature** that would have been difficult to add/maintain within the real\_time.pl script (because it contains [several real time tools](http://bdrouvot.wordpress.com/2013/01/30/real-time-database-utilities-grouped-into-a-single-script/ "Real-Time database utilities grouped into a single script")).

The result of this extraction is a new script called **asm\_metrics.pl** that** **can be downloaded from [this repository.](https://docs.google.com/folder/d/0B7Jf_4JdsptpRHdyOWk1VTdUdEU/edit)

<span style="text-decoration:underline;">**Not speaking about the new feature yet, the asm\_metrics.pl provides the same functionalities with the same options as my "deprecated" asmiostat one. That is to say:**</span>

<span style="text-decoration:underline;">It displays the following metrics:</span>

-   Reads/s: Number of read per second.
-   KbyRead/s: Kbytes read per second.
-   Avg ms/Read: ms per read in average.
-   AvgBy/Read: Average Bytes per read.
-   Same metrics are provided for Write Operations.

<span style="text-decoration:underline;">In such a way:</span>

-   It takes a snapshot each second (default interval) from the gv$asm\_disk\_iostat (or gv$asm\_disk\_stat depending of the version) cumulative view and **computes the delta** with the previous snapshot.
-   It allows to **display/aggregate/filter following your needs** on [ASM instances, diskgroup, failgroup](http://bdrouvot.wordpress.com/2013/02/15/asm-io-statistics-utility/ "ASM I/O Statistics Utility") (or [exadata cell's IP](http://bdrouvot.wordpress.com/2013/03/06/asm-io-statistics-utility-update-for-exadata/ "ASM I/O Statistics Utility: Update for Exadata")), disks and on [database instances](http://bdrouvot.wordpress.com/2013/07/05/asm-io-statistics-utility-v2/ "ASM I/O Statistics Utility V2").
-   It allows to sort based on the [number of reads/s, number of writes/s](http://bdrouvot.wordpress.com/2013/07/05/asm-io-statistics-utility-v2/ "ASM I/O Statistics Utility V2") or [both (iops)](http://bdrouvot.wordpress.com/2013/09/05/find-out-the-most-physical-io-consumers-through-asm-in-real-time/ "Find out the most physical IO consumers through ASM in real time").

<span style="text-decoration:underline;">So that, for example:</span>

-   You can find out which [databases are the most physical IO consumers](http://bdrouvot.wordpress.com/2013/09/05/find-out-the-most-physical-io-consumers-through-asm-in-real-time/ "Find out the most physical IO consumers through ASM in real time") (reads, writes or both).
-   You can find out which diskgroup is the most responsible for the physical IO (reads,writes or both).
-   You can see the [ASM preferred read in action](http://bdrouvot.wordpress.com/2013/05/05/asm-preferred-read-collect-performance-metrics-thanks-to-my-amsiostat-utility-and-slob-2/ "ASM Preferred Read: Collect performance metrics thanks to my amsiostat utility and SLOB 2") or [not](http://bdrouvot.wordpress.com/2013/07/02/flex-asm-12c-12-1-and-extended-rac-be-careful-to-unpreferred-read/ "Flex ASM 12c (12.1) and Extended Rac: be careful to “unpreferred” read !").
-   You can also find out…. (I let you finish the sentence as the asm\_metrics.pl utility output is customizable: see this [post](http://bdrouvot.wordpress.com/2013/02/15/asm-io-statistics-utility/ "ASM I/O Statistics Utility")).

<span style="text-decoration:underline;">Let's see the help and I'll explain the new feature:  
</span>

    ./asm_metrics.pl -help

    Usage: ./asm_metrics.pl [-interval] [-count] [-inst] [-dbinst] [-dg] [-fg] [-ip] [-show] [-display] [-sort_field] [-help]

     Default Interval : 1 second.
     Default Count    : Unlimited

      Parameter         Comment                                                           Default
      ---------         -------                                                           -------
      -INST=            ALL - Show all Instance(s)                                        ALL
                        CURRENT - Show Current Instance
                        INSTANCE_NAME,... - choose Instance(s) to display

      -DBINST=          Database Instance to collect (Wildcard allowed)                   ALL
      -DG=              Diskgroup to collect (comma separated list)                       ALL
      -FG=              Failgroup to collect (comma separated list)                       ALL
      -IP=              IP (Exadata Cells) to collect (Wildcard allowed)                  ALL
      -SHOW=            What to show: inst,dbinst,fg|ip,dg,dsk (comma separated list)     DG
      -DISPLAY=         What to display: snap,avg (comma separated list)                  SNAP
      -SORT_FIELD=      reads|writes|iops                                                 NONE

    Example: ./asm_metrics.pl
    Example: ./asm_metrics.pl  -inst=+ASM1
    Example: ./asm_metrics.pl  -dg=DATA -show=dg
    Example: ./asm_metrics.pl  -dg=data -show=inst,dg,fg
    Example: ./asm_metrics.pl  -show=dg,dsk
    Example: ./asm_metrics.pl  -show=inst,dg,fg,dsk
    Example: ./asm_metrics.pl  -interval=5 -count=3 -sort_field=iops
    Example: ./asm_metrics.pl  -show=dg -display=avg -sort_field=iops
    Example: ./asm_metrics.pl  -show=dg -display=snap,avg -sort_field=iops

<span style="text-decoration:underline;">**The "-display" option is the new one:**</span> It allows you to display the delta snap values (as previously), the average delta values since the collection began (that is to say since the script has been launched) or both.

**<span style="text-decoration:underline;">Let's see one use case:</span>**

I want to find out the most physical IO consumers for the DATA diskgroup every 5 seconds **and** since I launched the script .

We just need to launch the script that way (using the -display option as I want to see the snap **and** average sections):

    ./asm_metrics.pl -show=dbinst -display=snap,avg -dg=DATA -interval=5 -sort_field=iops

So that after the first 5 seconds the output looks like:

    ............................
    Collecting 5 sec....
    ............................

    ......... SNAP TAKEN AT ...................

    22:53:26                                                                             Kby      Avg       AvgBy/               Kby       Avg        AvgBy/
    22:53:26   INST     DBINST        DG            FG            DSK          Reads/s   Read/s   ms/Read   Read      Writes/s   Write/s   ms/Write   Write
    22:53:26   ------   -----------   -----------   -----------   ----------   -------   ------   -------   ------    ------     -------   --------   ------
    22:53:26            BDTO_1                                                 5.8       92.8     0.9       16384     2.4        38.4      1.6        16384
    22:53:26            BDTO_2                                                 5.0       80.0     1.6       16384     0.8        12.8      4.7        16384
    22:53:26            JCAASM_2                                               3.4       54.4     0.2       16384     2.4        38.4      2.0        16384
    22:53:26            JCAASM_1                                               2.4       38.4     1.5       16384     0.8        12.8      6.0        16384
    22:53:26            MILASM_1                                               2.4       38.4     0.6       16384     0.8        12.8      1.7        16384
    22:53:26            IATEBDTO_1                                             1.6       25.6     1.7       16384     0.4        6.4       5.0        16384
    22:53:26            SMTBDTO_2                                              0.0       0.0      0.0       0         0.0        0.0       0.0        0

    ......... AVERAGE SINCE ...................

    22:53:26                                                                             Kby      Avg       AvgBy/               Kby       Avg        AvgBy/
    22:53:26   INST     DBINST        DG            FG            DSK          Reads/s   Read/s   ms/Read   Read      Writes/s   Write/s   ms/Write   Write
    22:53:26   ------   -----------   -----------   -----------   ----------   -------   ------   -------   ------    ------     -------   --------   ------
    22:53:26            BDTO_1                                                 5.8       92.8     0.9       16384     2.4        38.4      1.6        16384
    22:53:26            BDTO_2                                                 5.0       80.0     1.6       16384     0.8        12.8      4.7        16384
    22:53:26            JCAASM_2                                               3.4       54.4     0.2       16384     2.4        38.4      2.0        16384
    22:53:26            JCAASM_1                                               2.4       38.4     1.5       16384     0.8        12.8      6.0        16384
    22:53:26            MILASM_1                                               2.4       38.4     0.6       16384     0.8        12.8      1.7        16384
    22:53:26            IATEBDTO_1                                             1.6       25.6     1.7       16384     0.4        6.4       5.0        16384
    22:53:26            SMTBDTO_2                                              0.0       0.0      0.0       0         0.0        0.0       0.0        0

So, no differences between the delta snap and the average after the first snap (which is obvious :-)) and as you can see the BDTO\_1 instance is the one that generated the most iops.

After 10 seconds the output looks like:

    ............................
    Collecting 5 sec....
    ............................

    ......... SNAP TAKEN AT ...................

    22:53:31                                                                             Kby      Avg       AvgBy/               Kby       Avg        AvgBy/
    22:53:31   INST     DBINST        DG            FG            DSK          Reads/s   Read/s   ms/Read   Read      Writes/s   Write/s   ms/Write   Write
    22:53:31   ------   -----------   -----------   -----------   ----------   -------   ------   -------   ------    ------     -------   --------   ------
    22:53:31            BDTO_2                                                 2.6       41.6     0.7       16384     0.8        12.8      1.5        16384
    22:53:31            JCAASM_2                                               3.0       48.0     0.2       16384     0.4        6.4       1.4        16384
    22:53:31            IATEBDTO_1                                             2.4       38.4     1.2       16384     0.8        12.8      2.2        16384
    22:53:31            JCAASM_1                                               2.2       35.2     1.3       16384     0.8        12.8      1.3        16384
    22:53:31            BDTO_1                                                 2.0       32.0     0.3       16384     0.4        6.4       1.4        16384
    22:53:31            MILASM_1                                               1.8       28.8     1.8       16384     0.4        6.4       2.5        16384
    22:53:31            SMTBDTO_2                                              0.0       0.0      0.0       0         0.0        0.0       0.0        0

    ......... AVERAGE SINCE ...................

    22:53:26                                                                             Kby      Avg       AvgBy/               Kby       Avg        AvgBy/
    22:53:26   INST     DBINST        DG            FG            DSK          Reads/s   Read/s   ms/Read   Read      Writes/s   Write/s   ms/Write   Write
    22:53:26   ------   -----------   -----------   -----------   ----------   -------   ------   -------   ------    ------     -------   --------   ------
    22:53:26            BDTO_1                                                 3.9       62.4     0.8       16384     1.4        22.4      1.5        16384
    22:53:26            BDTO_2                                                 3.8       60.8     1.3       16384     0.8        12.8      3.1        16384
    22:53:26            JCAASM_2                                               3.2       51.2     0.2       16384     1.4        22.4      1.9        16384
    22:53:26            JCAASM_1                                               2.3       36.8     1.4       16384     0.8        12.8      3.6        16384
    22:53:26            MILASM_1                                               2.1       33.6     1.1       16384     0.6        9.6       2.0        16384
    22:53:26            IATEBDTO_1                                             2.0       32.0     1.4       16384     0.6        9.6       3.1        16384
    22:53:26            SMTBDTO_2                                              0.0       0.0      0.0       0         0.0        0.0       0.0        0

As you can see the **delta snap section reports the time of the snap while the average section reports the time when the collection began**. By the way you can also see that the BDTO\_1 database instance is the one that generated the most IOPS since the collections began (while this is the BDTO\_2 one that generated the most during the second snap).

<span style="text-decoration:underline;">**Remarks:**</span>

-   The script does not create any objects, simply download it and use it !
-   If you hit this issue:

<!-- -->

    ./asm_metrics.pl 
    : No such file or directory

-   Then launch it that way:

<!-- -->

    perl ./asm_metrics.pl

-   It will not be possible to launch the "deprecated" asmiostat through the real\_time.pl anymore: Instead a warning message will be displayed to reflect this change and a link to the new asm\_metrics.pl script will be provided.

<span style="text-decoration:underline;">**Conclusion:**</span>

-   The asm\_metrics.pl script has been extracted from the real\_time.pl one.
-   It provides exactly the same features.
-   It also allows to display the delta snap values (by default), the average delta values since the collection began or both.
