---
layout: post
excerpt: "Intro into CCW Estimate API"
title: Intro into CCW Estimate API
categories: [API, CCW, DevNet]
author: Matyas
published: true
comments: true
---

### Intro
I work in pre-sales so working with CCW takes up a big part of my working day (joy!) and as a network automation enthusiast I've decided to automate the heck out of CCW. After a few people asked me to help them with CCW Estimate APIs, I decided to document the process so it can, hopefully, help others.

### What is CCW?
CCW stands for Cisco Commerce Workspace. If you work in pre-sales or sales, you probably spend a significant amount of your time in CCW. CCW allows you to create BoMs, subscriptions, quotes, deal IDs and orders of Cisco equipment. As I mentioned above, I work in pre-sales so I use CCW Estimate mainly for building BoMs, and that will be the focus of a few of my upcoming blog posts.

### Where to start?
You won't find a CCW sandbox on DevNet or any community forum where you could ask questions concerning CCW. A lot of the stuff that I'll be talking about here is therefore stuff I learnt through trial and error.

Before we actually start talking about CCW Estimate APIs, the first thing you should do is to set up a ClientID and Client Secret as these will be necessary for authentication.

### Requesting your ClientID and Client Secret
Let’s start. You will need to login with your Cisco login at [https://apiconsole.cisco.com/](https://apiconsole.cisco.com/). Click on ‘My Apps & Keys’ and ‘Register a New App’.

&nbsp;
<img src="../../img/2020-03-18-ccw-api-intro/my-apps-and-keys.png"
     alt="Postman Auth" width="500" align="center" />

When you are in the ‘Register a New App’ section, you will need to fill in some of the details relating to your new application. There are two important settings: 'OAuth2.0 Credentials', which needs to be configured as ‘Resource Owner Credentials’, and ‘Select APIs’ where you need to choose ‘CCW Estimate API’. This will generate a ClientID and Client Secret specific to this API.

&nbsp;
<img src="../../img/2020-03-18-ccw-api-intro/register-new-app.png"
     alt="Postman Auth" width="500" align="center" />

Shortly after this you should receive an email with your ClientID and Client Secret.

### CCW API Requests
CCW API Requests are very specific and a bit different from typical API requests, which you issue with respect to Cisco hardware or software.
Firstly, CCW API requests use OAuth2.0 for authentication. For those who are not familiar with OAuth2.0 for this type of authentication, you will need a username, password and also a ClientID and Client Secret. I described how you can obtain a ClientID and Client Secret in the previous section.

Secondly, CCW API requests are all POST requests. Even if you query CCW API you are still carrying out a POST request.

Last but not least, if you need to send the body you use XML. For me, the body was the most challenging part in the entire process. CCW API is very sensitive about the use of the correct syntax and, as the relevant documentation can be a bit tricky, I would like to focus mainly on this in this post. There will be lots of XML parsing so wait for it! This is going to be so much fun:)

CCW API response is in the XML format so it can be a little problematic as well but I will help you to understand how this can be done. I’m of course not saying that my way is the only, or best, way. I would love to hear anyone's suggestions for parsing data from an XML response, though I intend to focus more on this in my next post.


### Authentication

You normally login to CCW with your CCO user account and you will use the same credentials for CCW Estimate APIs. CCW APIs use OAuth2.0 for authentication; you will therefore need a ClientID and a Client Secret on top of your username and password. I explained how you can get these details from the CCW API portal in the previous section.

Let’s say we have all the details we need: a CCO username and password, Client ID and Client Secret. We will now therefore need to obtain the token, which we'll use for CCW API requests. For testing purposes, I would suggest starting with Postman to test that you have the right credentials and privileges. You can find exported Postman collections in the git [repo](https://git.matyasprokop.com/mprokop/ccw-tools-blog-repo){:target="blank"} in directory `postman`. Import the JSON file into your Postman and you should have 5 basic CCW API requests: AcqEstimate, createEstimate, getProduct, listEstimate and updateEstimate.

URLs which we will be using for three basic API call:

* List Estimate request - `https://api.cisco.com/commerce/EST/v2/async/listEstimate`
* Update Estimate request - `https://api.cisco.com/commerce/EST/v2/async/updateEstimate`
* Create Estimate request - `https://api.cisco.com/commerce/EST/v2/async/createEstimate`

I would suggest that you start with listEstimate as it is the easiest one. listEstimate request will query the address `https://api.cisco.com/commerce/EST/v2/async/listEstimate` and it will list all the estimates available.

ListEstimate will run a POST API request to the URL `https://api.cisco.com/commerce/EST/v2/async/listEstimate` but let’s check the authorization first. Choose OAuth2.0 in authorization of the request and click on ‘Get New Access Token’. The below window should open:
&nbsp;
<img src="../../img/2020-03-18-ccw-api-intro/postman-auth.png"
     alt="Postman Auth" width="500" align="center" />

When you click ‘Request Token’ you should obtain the token. Click on ‘Use Token’. Authorization will be exactly the same with the other CCW API requests as well.

### Skeleton body of CCW API Request
Before we will click on ‘Send’, we should check what it is we're actually sending in the body of the request. The body is always in the XML format wrapped in something called `soapenv`. `soapenv` stands for SOAP Envelope and it helps to indicate the start and the end of the message so that the receiver knows when an entire message has been received. Thanks to SOAP Envelope you know when you are done receiving a message and this is ready to be processed. SOAP Envelope is therefore basically a packaging mechanism.

When you click on body in Postman, you will see this:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hs="https://api.cisco.com/commerce/EST/v2/async/listEstimate">
 <soapenv:Body>
 <GetQuote releaseID="2014" versionID="1.0" systemEnvironmentCode="Production" languageCode="en-US" xmlns="http://www.openapplications.org/oagis/10">
 	 <ApplicationArea>
 	  <Sender>
       <ComponentID schemeAgencyID="Cisco">B2B-3.0</ComponentID>
    </Sender>
 	 </ApplicationArea>
     <DataArea>
		    <Get maxItems="25">
		      <Expression expressionLanguage="SortBy">LAST_MODIFIED</Expression>
		    </Get>
	 </DataArea>
 </GetQuote>
 </soapenv:Body>
</soapenv:Envelope>
```

You will see in further sections that the body of the request always starts and ends with `<soapenv:Envelope>` and `</soapenv:Envelope>`. `<ComponentID schemeAgencyID="Cisco">B2B-3.0</ComponentID>` is also an important part. If you use a different schemeAgencyID value, the request will not be valid.

`$xmlns:hs` needs to be defined based on the type of the request:

* List Estimate request - `https://api.cisco.com/commerce/EST/v2/async/listEstimate`
* Update Estimate request - `https://api.cisco.com/commerce/EST/v2/async/updateEstimate`
* Create Estimate request - `https://api.cisco.com/commerce/EST/v2/async/createEstimate`

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hs="$xmlns">
 <soapenv:Body>
 <GetQuote releaseID="2014" versionID="1.0" systemEnvironmentCode="Production" languageCode="en-US" xmlns="http://www.openapplications.org/oagis/10">
 	 <ApplicationArea>
 	  <Sender>
       <ComponentID schemeAgencyID="Cisco">B2B-3.0</ComponentID>
    </Sender>
 	 </ApplicationArea>
     <DataArea>

     ........

	 </DataArea>
 </GetQuote>
 </soapenv:Body>
</soapenv:Envelope>
```

### Skeleton body of CCW API Response
Now that we know what the authorization and the request body look like, we will have a look at the structure of the response. As I've mentioned before, the response is structured in the XML format.

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Body>
        <AcknowledgeQuote xmlns="http://www.openapplications.org/oagis/10">
            <ApplicationArea>
                <CreationDateTime>2020-03-19T08:17:37Z</CreationDateTime>
                <BODID schemeAgencyID="Cisco" schemeVersionID="1.0">3b89bde2-348f-4630-96db-22fd9c10af02</BODID>
            </ApplicationArea>
            <DataArea>
                ........
            </DataArea>
        </AcknowledgeQuote>
    </soapenv:Body>
</soapenv:Envelope>
```

You can see that the body of the response is very similar to the request, however there are some minor differences. For instance `soapenv:Body` contains `AcknowledgeQuote` and not the `GetQuote` attribute.

### Summary
Let’s summarise what we have learned so far.
CCW API requests and responses are encoded in the XML format. We've gone through, step by step, how to get a ClientID and Client Secret, which are necessary for CCW API authentication. We have showed that each request is wrapped in SOAP Envelope and explained some of its benefits. CCW requests are always sent using the POST method via REST API.

In the next post I will focus on specific examples of how to create, list and read CCW estimates. Stay tuned :)

### Sources
[https://apiconsole.cisco.com](https://apiconsole.cisco.com){:target="blank"}<br>
[https://apiconsole.cisco.com/docs/read/external_apis/CCW_Estimate_API](https://apiconsole.cisco.com/docs/read/external_apis/CCW_Estimate_API){:target="blank"}<br>
[https://git.matyasprokop.com/mprokop/ccw-tools-blog-repo](https://git.matyasprokop.com/mprokop/ccw-tools-blog-repo){:target="blank"}<br>
