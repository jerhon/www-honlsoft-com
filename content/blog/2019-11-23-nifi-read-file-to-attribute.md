+++ 
date = "2019-12-01"
title = "Apache Nifi: Reading a File Into an Attribute."
tags = ["nifi", "groovy"]
categories = ["blog"]
+++

I've been working with [Apache Nifi](https://nifi.apache.org/) for some integration projects at work. It's a decent data workflow tool. Consider it's free, then it's a great integration tool in businesses where cost or infrastructure management is an issue. It's also graphical, which personally I'm not a fan of (give me code any day), but lowers the bar to entry for simple problems and those less experienced in coding.

I recently had a problem that I thought would be simple for Nifi to solve. I had a set of [Apache Avro](https://avro.apache.org/) schemas stored as files used in a set of processors that convert XML to CSV. However, I didn't want to have to maintain these in Nifi as the flow was set up.  The flow had been written such that whenever a change was made, it had to be copied into a processor to apply the attribute on a flow file. I wanted them to be maintained in our on-prem GitLab instance to be close to some XSLT files that had been written as data conversions.

In the past the process being used was to modify the file, then copy and paste the file into a property on an UpdateAttribute processor to store the 'avro.schema'.  This had to be done across approximately 30 of these processors.  It was tedious, and things got out of sync quickly.

The files are already copied to a file share as part of a deployment process so that Nifi can access them.  I thought I could use an existing processor to read the file and store it as an attribute.  No luck... nothing fit the bill out of the box. Apache Nifi does have the ability to build out customer processors as part of it's data flow. However, to build one requires reving up a java project, storing it some where, building it, deploying it, etc.  Then maintaining it on every upgrade, etc. Yuck... Thankfully groovy came to the rescue. When a simple change is needed that isn't supported out of the box, writing a Groovy script can be an easy way around this.

Here's a listing of a groovy script that did what I needed.  It read the Avro schema from a file path, a mounted drive with the deployed schema on it, and puts it in an attribute on the flow file.

```groovy
def flowFile = session.get()

if (!flowFile) return
try {
    String filePath = flowFile.getAttribute("avroSchemaPath")
    log.debug(filePath);
    def file = new File(filePath)
    String fileContent = file.text
    log.debug(fileContent)
    flowFile.putAttribute("avro.schema", fileContent)

    session.transfer(flowFile, REL_SUCCESS)
} catch (e) {
    log.error('Failed to read avro schema from file.', e);
    session.transfer(flowFile, REL_FAILURE)
}
```

Much easier than the other alternatives.

There are a things to note here that I didn't know about when I started.  

* It's possible a session.get() may return without a flow file.  So, it is important to check the Flow File from session.get() is actually non-null.
* The flow file always needs to be transferred to success or failure at the end of the flow.  Especially for steps processing something and it's being passed on to the next step.

I wouldn't suggest usage of this for particularly complex programming tasks. However, for simple things such as this, it unlocks a lot of power.
