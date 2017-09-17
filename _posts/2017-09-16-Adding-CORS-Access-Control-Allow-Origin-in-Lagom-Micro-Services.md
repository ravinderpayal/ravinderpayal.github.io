---
layout: post
title: Adding CORS(Access-Control-Allow-Origin) in Lagom MicroServices.
author: ravinder_payal
---
<p>
Hi friends, today, I came accross an interesting challenge of adding CORS in Lagom MicroServices, but didn't find anything other than few google group threads. So, after completing the challenge I thought about sharing the working solution. So let's go ahead.
</p>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>


Adding CORS filter in Lagom is a few steps work.

For this we are going to use CORS filter provided by Play Framework as Lagom is also based on PlayFramework.(https://www.playframework.com/documentation/2.5.x/CorsFilter)

Credits: This google groups thread. https://groups.google.com/d/msg/lagom-framework/dtYN_1Ds4SQ/U3lzjwspBwAJ

Now come to the point, How to?

So, Step 1, go to build.sbt residing in the root folder containing the api and implementations of services. Now find the code-block having the following similar code, and add `filters` project dependencies.

>CorsFilter is a Play project, but we need not to explicitly add PlayFramework plugin in our plugins.sbt and plugin configuation in build.sbt as Lagom is based on Play, it consists all Play Projects.

```scala
lazy val `YOUR_SERVICE_NAME-impl` = (project in file("YOUR_SERVICE_NAME-impl"))
  .enablePlugins(LagomScala)
  .settings(
    libraryDependencies ++= Seq(
      .....,
      filters,//We have to add this
      ........
    )
  )
```
Now, we can use this filters project in our service.
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-format="fluid"
     data-ad-layout="in-article"
     data-ad-client="ca-pub-3201220427379470"
     data-ad-slot="2921542940"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

In step 2, we will change the Your_Service_NameApplication class which resides along with the class with service implementation.

So, go to the block having just mentioned class, and extend it with `CORSComponent` class. After this, in same code block, we have to add one more line of code for overiding the httpFilters and adding the CORS filter.

```scala
import play.filters.cors.CORSComponents//Import #1
import play.api.mvc.EssentialFilter//Import 2
abstract class WebtrackingApplication(context: LagomApplicationContext)
  extends LagomApplication(context)
    with .....
    with CORSComponents {//Step #2.a
....
  override lazy val httpFilters: Seq[EssentialFilter] = Seq(corsFilter)//Step #2.b
....
}
```
>Remember to import CORSComponent class and EssentialFilter type class.

<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-format="fluid"
     data-ad-layout="in-article"
     data-ad-client="ca-pub-3201220427379470"
     data-ad-slot="2921542940"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>


Now, Reload the project in SBT.

$>sbt

then
......>runAll

Cheers Guys. If you liked the post please share it in your group.
