---
layout: post
title: Reading Papers About Web Crawler
---

### Web Crawler--Wikipedia

> As Edwards et al. noted, "Given that the bandwidth for conducting crawls is neither infinite nor free, it is becoming essential to crawl the Web in not only a scalable, but efficient way, if some reasonable measure of quality of freshness is to be maintained." A crawler must carefully choose at each setp which pages to visit next.


> The behavior of a Web crawler is the outcome of a combination of policies:
> 
> * a *selection policy* that states which pages to download,
> * a *re-visit policy* that states when to check for changes to the pages,
> * a *politness policy* that states how avoid overloading Web sites, and
> * a *parallelization policy* that states how to coordinate(协调) distributed web crawlers. 

###### Selection policy
> As a crawler always downloads just a fraction of the Web pages, it is highly desirable that the downloaded fraction contains the most relevant pages and not just a random sample of the Web.
>
>This requires a metric of importance for prioritizing Web pages. The importance of a page is a function of its intrinsic quality, its popularity in terms of links or visits, and even of its URL(the latter is the case of vertical search engines restricted to a single top-level domain, or search engines restricted to a fixed Web site).

###### Restricting followed links
> A crawler may only want to seek out HTML pages and avoid all other MIME types. In order to request only HTML resources, a crawler may make an HTTP HEAD request to determine a Web resource's MIME type before requesting the entire resource with a GET request. To avoid making numerous HEAD requests, a crawler may examine the URL and only request a resource if the URL ends with certain characters such as .html, .htm, .asp, .aspx, .php, .jsp, .jspx or a slash. This strategy may cause numerous HTML Web resources to be unintentionally skipped.

###### URL normalization
>Crawlers usually perform some type of URL normalization in order to avoid crawling the same ersource more than once. The term *URL normalization*, also called *URL cannonicalization*, refers to the process of modifying and standardizing a URL in a consistent manner.
There are several types of normalization that may be performed including conversion of URLs to lowercase, removal of "." and ".." segments, and adding trailing slashes to the non-empty path component.

###### Re-visit policy
>The Web has a very dynamic nature, and crawling a fraction of the Web can take weeks or months. By the time a Web crawler has finished its crawl, many events could have happend, including creations, updates and deletions.
>From the search engine's point of view, there is a cost associated with not detecting an event, and thus having an outdated copy of a resource. The most-used cost functions are freshness and age.
>
> __Freshness__: This is a binary measure that indicates whether the local copy is accurate or not. The freshness of a page *p* in the repository at time *t* is defined as:
> ![](http://upload.wikimedia.org/math/0/7/2/072cc7b185ff9befceae814f19f232af.png)
>
> __Age__: This is a measure that indicates how outdated the local copy is. The age of a page *p* in the repository, at time *t* is defined as:
> ![](http://upload.wikimedia.org/math/a/5/8/a58f53c6251f60949b1cd8f63da43503.png)
>
>The objective of the crawler is to keep the average freshness of pages in its collection as high as possible, or to keep the average age of pages as low as possible. These objectives are not equivalent: in the first case, the crawler is just concerned with how many pages are out-dated, while in the second case, the crawler is concerned with how old the local copies of pages are.
>
>Tow simple re-visiting policies were studied by Cho and Garcia-Molina:
>
> __Unifom(均匀的) policy__: This involes re-visiting all pages in the collection with the same frequency, regardless of their rates of change.
> 
> __Proportional(成比例的) policy__: This involves re-visiting more often the pages that change more frequently. The visiting frequency is directly proportional to the (estimated) change frequency.
>
> (In both cases, the repeated crawling order of pages can be done either in a random or a fixed order.)
>
>Cho and Garcia-Molina proved the surprising result that, in terms of average freshness, the uniform policy outperforms the propotional policy in both a simulated Web and a real Web crawl. Intuitively(凭直觉), the reasoning is that, as web crawlers 