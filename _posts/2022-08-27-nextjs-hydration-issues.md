---
layout: post
title:  "Next JS Hydration Issues"
date:  2022-08-27 15:08
categories: NextJS
tags: NextJS React
---

In the past I was working on a NextJS site and after some time I have noticed some console errors being logged referring to NextJS hydration issues. This did not have any detrimental effect on the site, but I thought it was worth some investigation.

## Initial Error

The console error being logged was not very detailed:

![NextJS hydration error](/assets/images/NextJSHydrationInitialError.png)

The <a href="https://reactjs.org/docs/error-decoder.html/?invariant=425">
  link
</a> that is provided in the error shows the following detail:

![NextJS hydration error](/assets/images/NextJSHydrationDetailedError.png)

So what is this error telling us? Something is changing value between the server rendering the item, and then the item being displayed to the client. My initial thoughts were that it could be related to the auth part of the site, as if a user is logged in their name displays in the header.

This error was not visible when I was developing locally, so it was specific to the production environment... I needed more information.

## Investigation

To allow me to resolve the issue I needed to see exactly what content was not matching. The simplest way for me to do that was to change the production environment to build the site in development mode. This may be different depending on the hosting provider but all I had to do was change the YAML file to run *yarn dev* instead of *yarn start*.

When I did this I got the following detailed console error:

![NextJS hydration error](/assets/images/NextJSHydrationFullError.png)

Ah! The issue is time zone related, as the server defaults to UTC and the machine I am viewing the website on is set to GMT.

## Possible Resolutions

### Attempt 1

My first thought on fixing the issue was to explicitly set the time zone of the server to GMT. I could easily do this by setting the Node environment variable TZ to the GMT value of *Europe/London*.

Although this would fix my issue, it would not fix the issue for anybody viewing the website from a different time zone.

### Attempt 2

My second idea was to disable Server-Side rendering for the date/time sections in the site as this would be a global fix for everybody.

I was using moment to render the date/time, which looked like:

{% highlight javascript %}

{moment(props.article.created).format('dddd Do MMMM YYYY HH:MM ')} 

{% endhighlight %}

To allow me to disable Server-Side rendering I changed to using the react-moment component **&lt;Moment&gt;**. I imported it as follows:

{% highlight javascript %}

const Moment = dynamic(() => import('react-moment'), {
  ssr: false
})

{% endhighlight %}

This then allowed me to render the date/time section on the client-side only:

{% highlight html %}

<Moment 
    format='dddd Do MMMM YYYY HH:MM ' 
    date={props.article.created} 
/>

{% endhighlight %}

This resolved all hydration / mismatch related issues.