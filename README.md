The goal of this servlet is to overcome a feature lacking in Ivy resolution.  Specifically, only the IbiblioResolver in
Ivy understands the notion of Maven SNAPSHOTs in repositories.  Therefore, requests handled by the UrlResolver, like the
ivy.xml and the dependencies resolved therein, will not properly retrieve the most recent timestamped SNAPSHOT from a
Maven repo.  It will just look only for the SNAPSHOT by name, without consideration for the maven-metadata.xml.  This
filter will examine requests with SNAPSHOT in the filename and attempt to dispatch them to the proper file.  This should
not affect Maven resolves as they are smart enough to ask for the maven-metadata.xml first and request the proper
timestamped file.

This project creates a war for deployment to a web container.  Placement of the war in a Jetty server will deploy to the
root context.

Modify the bin/jetty.sh file of a Jetty installation to define a proxiedURL system property that points to the
web application context url of the repo you want to proxy.  You should also define a redirectURL system property
that points to the repo you are working with itself.  For instance, if you are running a Nexus instance on the root
context of ROOT on localhost:8081, you should add the following JAVA_OPTIONS line prior to the "start" definition

```
JAVA_OPTIONS+=("-DproxiedURL=http://localhost:8081" "-DredirectURL=http://nexus.pentaho.org")
```

Keep in mind, the only difference between proxiedURL and redirectURL is one of directness.  redirectURL is the front door
from the perspective of the client.  proxiedURL is how you are going to grab the bytes out of the repo, presumably bypassing
stuff that the folks on the outside need to go through to get there.  If you don't have a direct internal route, then the
two values could be the same.

You can also do some URL rewriting if you want to transpose to a different repository aggregate.  For example, let's say
that your requests are asking for something in content/groups/omni, but you want the requests to be routed to 
artifactory/omni.  In that case, you might specify your JAVA_OPTIONS as:

```
JAVA_OPTIONS+=("-DproxiedURL=http://localhost:8081" "-DredirectURL=http://10.177.177.134:8081" "-DurlSearchString=content/groups/omni" "-DurlReplaceString=artifactory/omni")
```

urlSearchString and urlReplaceString may also be comma separated ordered lists (no spaces) for doing multiple replacements.

It is also expected that your url resolver in your ivysettings.xml will contain checkmodified="true" and
changingPattern="*-SNAPSHOT".