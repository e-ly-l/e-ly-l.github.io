+++
title = "When Proxy Matters"
description = ""
summary = ""
date = 2024-09-23T23:45:00+08:00
slug = "when-proxy-matters"
tags = ["proxy"]
categories = ["implement"]
+++

## Proxy? VPN? What's that?

A VPN (Virtual Private Network) is a technology that creates a secure connection over a public network, such as the internet. A proxy, on the other hand, is a server that acts as an intermediary for requests from clients seeking resources from other servers.

In terms of network layers, a VPN operates at the network layer, while a proxy functions at the application layer. A VPN offers a more secure way to access the internet, whereas a proxy provides more flexibility for accessing specific resources.

## When to use Proxy or VPN

For personal use—such as browsing the web, streaming videos, and gaming, especially when on public Wi-Fi—I would choose a VPN. It provides better security and privacy. On the other hand, for business purposes like web scraping, data mining, or penetration testing, where I frequently need to change my IP address to avoid being blocked, I would opt for a proxy.

## Why I need Proxy

In my daily work, I scrape data from the internet, so we’ve built a proxy pool to avoid getting blocked by various platforms. This allows us to scrape data from millions of creators across multiple platforms every day.

However, our proxy pool is growing more slowly than our scraping demands. We also need to monitor the health of each proxy, but want to minimize the maintenance work required. Recently, I’ve been exploring proxy services to bridge the gap between our needs and available resources.

## Types of Proxy Resources

There are three main types of proxy resources: datacenter, residential, and mobile proxies. Datacenter proxies are the cheapest and fastest, but they are easy to detect and block. Residential proxies are more expensive and slower, with occasional disconnects, but they are harder to detect and block. Mobile proxies are the most expensive and slowest, but they are also the hardest to detect and block.

**Mobile > Residential > Datacenter**

This formula represents both the quality of proxies for scraping and their cost. Many proxies offer the SOCKS5 protocol, which is faster and more secure than the HTTP protocol.

After researching, we decided to use residential proxies for our business. They provide the best balance for our data scraping needs, and we chose a relatively affordable residential proxy service to stay within our budget.

## Interesting Proxy Services and their Features

While searching for proxy services, I found some interesting options. Here are a few of them:

### [Scraper API](https://www.scraperapi.com/)
Scraper API provides a [Scraping API](https://www.scraperapi.com/solutions/scraping-api/) that allows you to scrape any website without getting blocked, and includes CAPTCHA handling and automatic parsing (supporting Amazon, Google Search, and Google Shopping as of writing).

They also offer a [DataPipeline](https://www.scraperapi.com/solutions/data-pipeline/) service, which extracts data without requiring any code.

### [Oxylabs](https://oxylabs.io/)
Oxylabs offers very detailed [documentation](https://developers.oxylabs.io/) and APIs for a wide range of use cases.

### [Nimble](https://www.nimbleway.com/)
Their [Batch processing](https://docs.nimbleway.com/data-platform/web-api/batch-processing) approach seems like an interesting way to manage large-scale data scraping tasks.
