---
title: How to write a spider
---
# Overview

------
Most of us know what is spider, but not all of us know how to write a spider to do some interesting things,  especially when we are new bee for it. Here I will show you how to write high flexibility spider.

A spider should contain some necessary parts,

> * **Downloader**: Download the page from the internet, like Apache HttpClient.
> * **Processer**: Parse the downloaded page and discover the target link, we can use Jsoup as the HTML parser, using XPath, we can get the target field we need.
> * **Pipiline**: Pipeline to extract the results of the processing, including computing, persistence to the file, database and so on. 
> * **Scheduler**: The Scheduler is to manage the URLs to be crawled, as well as some heavy work.

[**Spider**](https://github.com/kekewang/Spider) is my first spider which is developed with Java, it depend on the spider framework named [WebMagic](http://webmagic.io/archive/docs/0.6.1/en/)

This project is to download the video data from the site Youku.com, firstly I hope I can get all the video data and store in db, so that I can do some analysis, like we can know the count of all the videos and the video category,  In fact, we can do that using webmagic easily,  but the youku Anti-spider will block your ip if your spider do the downloading frequently, so we have to do something which can confront with the anti-spider system.
```Java
public class YoukuProcesser implements PageProcessor {

    private Site site = Site.me().setRetryTimes(5).setSleepTime(1000).setTimeOut(10000);

    public List<Proxy> proxyList = new ArrayList();

    public BufferedReader proxyIpReader = new BufferedReader(new InputStreamReader(YoukuProcesser.class.getResourceAsStream("/config/proxyip.txt")));

    public static int pageCount = 0;

    public void process(Page page) {
        page.addTargetRequests(page.getHtml().links().regex("//v.youku.com/v_show/id_\\w+.*").all());
        page.putField("url", page.getUrl().toString());
        page.putField("title", page.getHtml().xpath("//div[@class='base_info']/h1[@class='title']/allText()").toString());
        page.putField("category", page.getHtml().xpath("//h1[@class='title']/a/text()").toString());
        page.putField("categoryUrl", page.getHtml().xpath("//h1[@class='title']/a/@href"));
        page.putField("vid", page.getUrl().regex("v.youku.com/v_show/id_([\\w+]*)==").toString());


        if (page.getResultItems().get("vid") == null) {
            //skip this page
            page.setSkip(true);
        }
        System.out.println("=========================Page No." + pageCount++ + "=========================");
    }

    public Site getSite() {
        return site;
    }

    public static void main(String[] args) {
        YoukuProcesser processer = new YoukuProcesser();
        HttpClientDownloader downloader = new HttpClientDownloader();
        downloader.setProxyProvider(processer.getSimpleProxyProvider());
        Spider.create(processer).addUrl("http://www.youku.com/").setDownloader(downloader).thread(100).run();
    }
}
```

# Proxy

------
Facing the blocking ip issue, what we should think about is how to change our ip after the spider fetch the page for a long time, but it seems that is unrealistic, we need get some proxy ip pool, xicidaili.com has some available proxy ip, and we can use spider to fetch all the ips and check the availability of them. After fetching enough available ips, we can fetch the video data easily.

```java
public class ProxyValidater {

    public static boolean checkproxy(String ip, String port) {
        try {
            InetAddress addr = InetAddress.getByName(ip);
            if (addr.isReachable(5000)) {
                return ensocketize(ip, Integer.parseInt(port));
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return false;
    }

    public static boolean ensocketize(String host, int port) {
        try {
            HttpClient client = new DefaultHttpClient();
            HttpGet get = new HttpGet("http://blanksite.com/");
            HttpHost proxy = new HttpHost(host, port);
            client.getParams().setParameter(ConnRoutePNames.DEFAULT_PROXY, proxy);
            client.getParams().setParameter(CoreConnectionPNames.SO_TIMEOUT, 15000);
            HttpResponse response = client.execute(get);
            HttpEntity enti = response.getEntity();
            if (response != null) {
                return true;
            }
        } catch (Exception ex) {
            System.out.println("Proxy failed");
        }

        return false;
    }

    public static void main(String[] args) {

        boolean result = ProxyValidater.checkproxy("125.118.65.122", "9999");

        System.out.println("Proxy validate result: " + result);
    }
}
```
When we get enough proxy ips, we can use proxy to fetch the pages.
