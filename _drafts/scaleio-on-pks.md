---
title: ScaleIO on PKS
date: 2018-08-01 00:00:00 -0700

---
Overall, Kubernetes is a stunningly extensible system - you can make it do almost anything you want.

However, that extensible nature is both a blessing and a curse - it can also be difficult to manage consistently at scale.  Pivotal attempts to address a significant part of that by using BOSH and the stemcells it deploys...it certainly helps.

However, it also adds some challenges - the same 'fixed' stemcells that provide simpler management also make it more difficult to make changes that impact the system.

I had a customer request to investigate what it would take to make VxFlexOS (nee ScaleIO) work with PKS.  VxFlexOS uses a kernel driver on the host to add block devices in `/dev` in the form of `/dev/sciniX`, which means editing the underlying stemcell to install the kernel driver (`scini.ko)` as well as the userspace control binary `drv_cfg`.  

Fortunately, BOSH has a tool for adding custom additions to a stemcell without modifying the original upstream source (which is nice for minimizing the footprint of things you need to manage/update).   We call these BOSH addons, I'd I recently updated some work done by Gary White and Paul Blum from the EMC/Pivotal teams to work with modern VxFlexOS and PKS.  I worked with a colleague Stu to build an[ updated version of such a release and posted it]().