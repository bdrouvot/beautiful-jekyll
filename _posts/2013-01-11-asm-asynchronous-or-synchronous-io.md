---
layout: post
title: ASM Asynchronous or Synchronous I/O
date: 2013-01-11 17:00:14.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ASM
tags: [ASM, oracle]
meta:
  _edit_last: '40807211'
  _publicize_pending: '1'
  publicize_twitter_user: BertrandDrouvot
  _wpas_done_2225791: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:246399475;b:1;}}
  _wpas_done_2077996: '1'
  _wpas_skip_2225791: '1'
  _wpas_skip_2077996: '1'
author:
  login: bdrouvot
  email: bdtoracleblog@gmail.com
  display_name: bdrouvot
  first_name: ''
  last_name: ''
permalink: "/2013/01/11/asm-asynchronous-or-synchronous-io/"
---

As you know ASM is doing non (operating system) buffered I/O (also known as ‘DIO’ or Direct I/O) regardless of the oracle database **filesystemio\_options** parameter.

<span style="color:#0000ff;">But what's about :  Asynchronous/Synchronous I/O ?</span>

If you have a look to [MOS note \[ID 751463.1\]](https://support.oracle.com/epmos/faces/ui/km/SearchDocDisplay.jspx?_afrLoop=638224979179169&recommended=true&type=DOCUMENT&id=751463.1&_afrWindowMode=0&_adf.ctrl-state=e23gwbnmc_184) you'll see that ASM asynchronous/synchronous I/O is entirely controlled by the DISK\_ASYNCH\_IO parameter and not the FILESYSTEMIO\_OPTIONS one.

At the time being, this note only deals with 10.2 databases, so I want to check if this is still the case with 11.2 databases (Let me tell you than I hope so ;-) ) :

<span style="text-decoration:underline;"><span style="color:#0000ff;text-decoration:underline;">For this test I will (at the database level):</span></span>

-   create a tablespace of 10M (<span style="color:#003300;">create tablespace bdt datafile '+DATA' size 10m</span>)
-   strace the DBW process (<span style="color:#003300;">strace -cp &lt;pid of dbw process&gt;</span>)

With differents values for **filesystemio\_options** and **disk\_asynch\_io**.

<span style="text-decoration:underline;color:#0000ff;">The results are :</span>

    filesystemio_options=setall
    disk_asynch_io=true

    % time seconds usecs/call calls errors syscall

    ------ ----------- ----------- --------- --------- ----------------
    41.39 0.001000 7 150 14 semtimedop
    35.02 0.000846 7 126 io_submit
    20.70 0.000500 4 126 io_getevents

As you can see the **io\_getevents** and **io\_submit** system calls have been used =&gt; **Asynchronous** I/O.

    filesystemio_options=setall
    disk_asynch_io=false

    % time seconds usecs/call calls errors syscall
    ------ ----------- ----------- --------- --------- ----------------
    75.38 0.007469 30 252 pwrite
    20.19 0.002000 10 202 5 semtimedop
    0.00 0.000000 0 8 close
    0.00 0.000000 0 2 2 stat

As you can see the **pwrite** system call has been used =&gt; **Synchronous** I/O.

    filesystemio_options=none
    disk_asynch_io=true

    % time seconds usecs/call calls errors syscall
    ------ ----------- ----------- --------- --------- ----------------
    75.45 0.005055 40 126 io_submit
    14.93 0.001000 6 157 2 semtimedop
    9.10 0.000610 5 126 io_getevents

As you can see the **io\_getevents** and **io\_submit** system calls have been used =&gt; **Asynchronous** I/O.

    filesystemio_options=none
    disk_asynch_io=false

    % time seconds usecs/call calls errors syscall
    ------ ----------- ----------- --------- --------- ----------------
    56.54 0.005354 21 252 pwrite
    42.23 0.003999 16 256 3 semtimedop
    0.51 0.000048 0 268 semctl
    0.49 0.000046 0 817 times

As you can see the **pwrite** system call has been used =&gt; **Synchronous** I/O.

<span style="text-decoration:underline;"><span style="color:#0000ff;text-decoration:underline;">Conclusion :</span></span>

With 11.2 databases, ASM asynchronous/synchronous I/O is still entirely controlled by the DISK\_ASYNCH\_IO parameter.
