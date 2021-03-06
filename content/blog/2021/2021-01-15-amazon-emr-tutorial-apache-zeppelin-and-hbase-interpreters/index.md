---
layout: post
title: "Amazon EMR tutorial: Apache Zeppelin and HBase interpreters"
date: 2021-01-15
comments: true
author: Rackspace Onica Team
published: true
authorIsRacker: true
categories:
    - AWS
metaTitle: "Amazon EMR tutorial: Apache Zeppelin and HBase interpreters"
metaDescription: "Amazon Elastic MapReduce (EMR) is an easier alternative to running in-house cluster computing.
By providing an expandable, low-configuration service, EMR simplifies the process of spinning up and maintaining
Hadoop and Spark clusters running in the cloud."
ogTitle: "Amazon EMR tutorial: Apache Zeppelin and HBase interpreters"
ogDescription: "Amazon Elastic MapReduce (EMR) is an easier alternative to running in-house cluster computing. By
providing an expandable, low-configuration service, it simplifies the process of spinning up and maintaining
Hadoop and Spark clusters running in the cloud."
slug: "amazon-emr-tutorial-apache-zeppelin-and-hbase-interpreters"
canonical: https://onica.com/blog/data-analytics/amazon-emr-tutorial/
---

*Originally published in Oct 2016, at Onica.com/blog*

We use Amazon EMR&reg; heavily for both customer projects and internal use-cases when we need to crunch huge
datasets in the cloud.

<!--more-->

### What is Amazon EMR?

Amazon&reg; Elastic MapReduce&reg; (EMR) is an easier alternative to running in-house cluster computing. By
providing an expandable, low-configuration service, it simplifies the process of spinning up and maintaining
Hadoop&reg; and Spark&reg; clusters running in the cloud. This drastically lowers the barriers of entry for
data teams to get started and allows users to easily and cost-effectively process huge amounts of data.

#### How does AWS EMR work?

Amazon EMR uses a hosted Hadoop framework as its data processing engine, running on the web-scale
infrastructure of Amazon Elastic Compute Cloud&reg; (Amazon EC2) and Amazon Simple Storage Service&reg;
(Amazon S3). By using the MapReduce programming model, Hadoop divides data into small fragments of work
that any cluster node can execute. Instead of using one large computer to store and process the data,
Hadoop can cluster many computers and crunch huge datasets much faster.

We have found increased interest from internal and external data teams wanting to collaborate by using
notebooks. Apache Zeppelin&reg; is an open-sourced project for web-based interactive notebooks for data
analytics. Typical use-cases include data ingestion, discovery, analytics, visualization, and collaboration.
Fortunately, Amazon EMR comes with Apache Zeppelin support baked right in. At the time of writing, Amazon EMR 5.0.0
supports Zeppelin version 0.6.1.

Besides, Amazon EMR supports Apache HBase (1.2.2) and Apache Phoenix (4.7.0) natively. HBase is a highly reliable
NoSQL store built upon the Hadoop Distributed File System, and Phoenix is a JDBC front end to the HBase engine, which
converts standard SQL into native HBase scans and queries. This enables a powerful point-lookup use-case, capable of
returning small results from billions of rows in milliseconds, or larger queries in seconds, by using standard SQL.
Finally, the HBase and Phoenix combo supports full ACID transactions, enabling OLTP workloads to run in a highly
available, scale-out architected database in AWS&reg;.

Of course, we wanted to see the power of using HBase and Phoenix for ourselves on EMR using Zeppelin notebooks,
but we identified a few extra steps to get this running. We hope we can *spark*&mdash;pun intended&mdash;your interest
in exploring big data sets in the cloud, by using EMR and Zeppelin.

### How do you set up an EMR? (An Amazon EMR Tutorial)

Following is the summary of the steps that we used to experiment with Zeppelin, Phoenix, and HBase on Amazon EMR:

1. Start an EMR cluster with Zeppelin, Phoenix, and HBase pre-configured.
2. SSH and web proxy into the EMR Master Node.
3. Install and configure Zeppelin interpreters for HBase and Phoenix.
4. Load data into HBase.
5. Query the data using Phoenix in Zeppelin to create charts and graphs.
6. Terminate the cluster.

Prerequisites:

- Familiarity with foundational AWS Concepts.
- An AWS account.
- A VPC with a public subnet, in US East N. Virginia.
- Basic knowledge of bash, \*nix command line, and optional knowledge of SQL.

#### 1. Start an EMR cluster with Zeppelin, Phoenix, and HBase pre-configured

Click [here](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=Zeppelin-Phoenix-Blog&templateURL=https:%2F%2Fs3-us-west-2.amazonaws.com%2Fcis-samples%2Fzeppelin-demo%2Fcfn%2Fzeppelin-demo.json)
to start an EMR cluster in your own AWS account by using the JSON CloudFormation&reg; template we provide. Ensure you
are currently logged in to an AWS account in your web browser, otherwise the link won't work.

This link gets you quickly started with provisioning the EMR cluster by using a JSON CloudFormation template. Follow
the prompts in the CloudFormation page to build the cluster. After it finishes, make a note of the table in the **Outputs**
tab as you will need the URLs later.

Perform the following steps:

a. Create the stack.

{{<img src="picture1.png" title="" alt="">}}

b. Specify the parameters with your VPC, subnet, and EC2 key.

{{<img src="picture2.png" title="" alt="">}}

c. Optionally, specify some tags.

{{<img src="picture3.png" title="" alt="">}}

d. Click **Create**.

{{<img src="picture4.png" title="" alt="">}}

e. Take a note of the **Outputs** tab.

{{<img src="picture5.png" title="" alt="">}}

#### 2. SSH and web proxy into the master node.

Perform the following steps to set yourself up so you can connect to the master EMR node through SecureShell (SSH):

a. Go to the EMR page and select your cluster.

{{<img src="picture6.png" title="" alt="">}}

b. Scroll down until you find **Security Groups for Master** and click the hyperlink “sg-********” in the console.

{{<img src="picture7.png" title="" alt="">}}

c. Select the **ElasticMapReduce-master** Security Group, click **Inbound**, and click **Edit**.

{{<img src="picture8.png" title="" alt="">}}

d. Add your IP by clicking **Add Rule**. Then, select **SSH** from the drop-down on the far right, select **My IP**
   in the second to last drop-down menu, and click **Save**.

{{<img src="picture9.png" title="" alt="">}}

e. SSH into the master node from your local workstation. You can find the steps on how to SSH into the master node
   by going back to your EMR cluster page (Step 2.a), and clicking the **SSH link** hyperlink.

{{<img src="picture10.png" title="" alt="">}}

{{<img src="picture10a.png" title="" alt="">}}

f. Finally, you need to set up a web connection via proxy to the EMR master node. You can find the steps by going
   back to your EMR cluster page (Step 2.a) and clicking the **Enable Web Connections** hyperlink.

{{<img src="picture11.png" title="" alt="">}}

#### 3. Install and configure Zeppelin interpreters for HBase and Phoenix

a. Start an SSH Terminal session (Step 2.e).

{{<img src="picture12.png" title="" alt="">}}

b. Run the following commands in sequence in the Terminal command-line interface:

            cd /usr/lib/zeppelin
            sudo bash bin/install-interpreter.sh -a
            sudo bash bin/zeppelin-daemon.sh restart

c. You have now changed directories into the **zeppelin** binaries folder, installed all possible interpreters,
   and restarted the service for the changes to take effect.

#### 4. Connect to the Zeppelin UI and set up the interpreters

a. Open a new Terminal window and start the SSH tunnel (Step 2.f).

{{<img src="picture13.png" title="" alt="">}}

b. Open your browser and enable your web proxy, such as FoxyProxy (Step 2.f).

{{<img src="picture14.png" title="" alt="">}}

c. Open your browser and point it to the URL in the *zeppelin* output from CloudFormation. You should see the following page:

{{<img src="picture15.png" title="" alt="">}}

d. Click the drop-down menu that says **anonymous** and select **Interpreter**.

{{<img src="picture16.png" title="" alt="">}}

e. Click **Create** in the top left corner.

{{<img src="picture17.png" title="" alt="">}}

f. Give the interpreter a name of `jdbc` and select **jdbc** from the **Interpreter** group drop-down list.

{{<img src="picture18.png" title="" alt="">}}

g. Scroll down in the **Properties** section until you see **phoenix** settings. Adjust the **phoenix.url** setting
   to: `jdbc:phoenix:localhost:8765/hbase`.

{{<img src="picture19.png" title="" alt="">}}

h. Scroll down to the **Dependencies** section, add the artifact `org.apache.phoenix:phoenix-core:4.7.0-HBase-1.1`,
   and click **Save**.

{{<img src="picture20.png" title="" alt="">}}

#### 5. Query the data by using Zeppelin

At this point, you are now ready to create a new Zeppelin notebook and start loading and querying data.

a. Return to the Zeppelin homepage, click **Import Note**, click **Add from URL**, copy and paste the
following URL in the URL field. Click **Import Note** in the pop-up screen, using [https://gist.githubusercontent.com/laithalsaadoon/
566407d2c0700f785eed87d5b73bdbf8/raw/9755ead50ab05e32e6c8af7bd170a09d49e90eb7/zeppelin-blog.json](https://gist.githubusercontent.com/laithalsaadoon/
566407d2c0700f785eed87d5b73bdbf8/raw/9755ead50ab05e32e6c8af7bd170a09d49e90eb7/zeppelin-blog.json).

{{<img src="picture21.png" title="" alt="">}}

{{<img src="picture22.png" title="" alt="">}}

{{<img src="picture23.png" title="" alt="">}}

b. Click the notebook that you just added.

{{<img src="picture24.png" title="" alt="">}}

c. You should see a few lines of code and scripts in a notebook similar to the following:

{{<img src="picture25.png" title="" alt="">}}

d. You can now hit the small **play** button in the top right corner of each white box&mdash;these are called *paragraphs*
   in Zeppelin vernacular.

{{<img src="picture26.png" title="" alt="">}}

e. Click **play** on the first paragraph&mdash;the one that includes **### Create an empty table in HBase**. Wait for it
   to say *FINISHED* in the top right corner before proceeding.

{{<img src="picture27.png" title="" alt="">}}

f. In the following paragraph, update the EMR endpoint URL on line 9 with your cluster’s corresponding URL.

g. Repeat the preceding *step d*, running each paragraph in sequence, waiting for each paragraph to say *FINISHED* before proceeding.

h. At the end, you should end up with a pie chart visualization similar to the following:

{{<img src="picture28.png" title="" alt="">}}

#### 6. Terminate the cluster

a. Click [here](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%253Ffilter%253Dactive%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=6ZgjASOTghiBI2OfYrxb7wz1eUKpb0B0UPQPaqbgljQ&code_challenge_method=SHA-256)
to return to the CloudFormation web page.

b. Select the template that you created as part of this demo.

{{<img src="picture29.png" title="" alt="">}}

c. From the **Actions** drop-down menu, select **Delete Stack**.

{{<img src="picture30.png" title="" alt="">}}

d. Click **Yes, Delete**.

{{<img src="picture31.png" title="" alt="">}}

### Conclusion

We hope you enjoyed our Amazon EMR tutorial on Apache Zeppelin and that it truly sparked your interest
in exploring big data sets in the cloud by using EMR and Zeppelin.

<a class="cta blue" id="cta" href="https://www.rackspace.com/cloud/aws">Learn more about Rackspace AWS services.</a>

Use the Feedback tab to make any comments or ask questions. You can also click **Sales Chat** to 
[chat now](https://www.rackspace.com/) and start the conversation.

