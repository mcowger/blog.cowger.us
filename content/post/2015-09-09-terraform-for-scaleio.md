---
layout: post
title: Deploying ScaleIO with Terraform
author: mcowger
categories: [terraform, scaleio, aws]
published: true
comments: true
---

Almost 2 years ago, shortly after EMC acquired ScaleIO, [I demonstrated the deployment of ScaleIO in AWS at significant scales](http://www.exaforge.com/scaleio-scale-200-nodes-and-beyond/) - up to 1000 nodes.  It was pretty fun and got some solid traction on [Chad's blog](http://virtualgeek.typepad.com/virtual_geek/2013/11/this-is-unbelievable-and-getting-to-the-point-of-the-ridiculously-awesome.html) and at an [EMCworld keynote](https://www.youtube.com/watch?v=lGj1xCvzPF8).

Now, that was cool, but I've been asked many times for how to replicate that.  While the code was open source, it wasn't very clean, and was quite fragile, and no one but me was able to get it working...and thats bad.

Also within the last 2 years, there are better tools out there.  Specifically, Hashicorp has released Terraform, which is software designed for building infrastructure on clouds.  I figured I would give it a try.

In just a day, I was able to get Terraform to deploy a fully working ScaleIO cluster on AWS hardware, in a VPC, and it does it in just a couple minutes.  You can [clone / fork the repo here](https://github.com/mcowger/terraform-scaleio) to try it out.

{% highlight shell-session  linenos=table %}

% git clone https://github.com/mcowger/terraform-scaleio.git    ~/Projects hex
Cloning into 'terraform-scaleio'...
remote: Counting objects: 15, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 15 (delta 5), reused 15 (delta 5), pack-reused 0
Unpacking objects: 100% (15/15), done.
Checking connectivity... done.

{% endhighlight %}

Thats easy enough, now lets set a couple settings (although the defaults work, and are minimal) by editing `variables.tf`:

{% highlight json linenos=table %}

variable access_key  {}
variable secret_key  {}
variable region  { default="us-east-1"}
variable ami_id  { default="ami-a8d369c0"}
variable key_name  {}
variable key_file  {default="~/.ssh/id_rsa"}
variable sds_count {default="3"}
variable sds_instance_type {default="r3.2xlarge"}
variable mdm_instance_type {default="m3.medium"}

{% endhighlight %}

Probably the thing you most care about changing is the `sds_count` and possible the `key_file`.  I'd recommend against changing the ami (its a RHEL7 AMI) and the instance types.

Save your file and boot the cluster:

{% highlight shell-session linenos=table %}
% terraform apply -var "access_key=XXXX" -var "secret_key=YYYY" -var "key_name=keyfile"
{% endhighlight %}

You'll see a BUNCH of output, but what it does is it goes out, creating an Amazon VPC, subnets, gateways, etc as needed.  It then builds the 3 (or more) SDS, installing the SDS on them.  Lastly, it installs and configures an MDM, adding the SDS's to the MDM's pool.  This should take no more than 3-4 minutes for a simple 3 node cluster, and less than 10 for a many-node cluster (no more than 250!).

When its finished, you'll see something like:

{% highlight shell-session linenos=table %}
Outputs:

  MDM_IP = 52.6.181.180
  MDM Password = admin/password123!
  SDS_IP = 10.0.104.250,10.0.180.242,10.0.94.144
{% endhighlight %}

You can SSH into the MDM using `ec2-user@MDM-IP` and your key file, and login to ScaleIO with `scli --login --mdm_ip MDMIP --username admin --password password123!`

Creating volumes and SDC's is an exercise (currently) left to the reader.

If you want to shut everything down, its as simple as:

{% highlight shell-session linenos=table %}
% terraform destroy -var "access_key=XXXX" -var "secret_key=YYYY" -var "key_name=keyfile"
{% endhighlight %}

All in all, with the defaults, this should cost you about $2.50 per hour to run.  Sweet.

As always, pull requests welcome!
