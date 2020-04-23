---
layout: post
title:  "Azure Resource Organization Cheat Sheet"
thumbnail: "/images/2020/azure_parts_thumb.png"
---

It's taken me some time, but I think I finally
have a handle on how all the various Azure resources are
organized and how exactly I'm paying for things.
I summarized what I know into a handy little cheat sheet
that I hope will help you if you're as confused as I was.

<a href="/images/2020/azure_parts.png"><img src="/images/2020/azure_parts.png" alt="Appstat screenshot" /></a>

<a href="/images/2020/azure_parts.pdf">Azure Resource Organization Cheat Sheet PDF</a>

## Account - Who are you?

Or, more specifically, what's your email address? Everything kicks off with a Microsoft account registered through one of their many services. You probably have one already.

## Active Directory - Who are you working with?

Active Directory is used for authentication - especially authentication for Microsoft services. One was probably created for you when you setup Azure with your Account. If you ever see all your Subscriptions or Services disappear, it's probably because you have selected the wrong AD.

## Subscription - How are you going to pay?

This is how you tell Microsoft how you're going to pay for all the goodies in the next sections. If you want to split Services over multiple payment methods, you're going to need multiple Subscriptions.

## Resource Group - How do you group things?

This is your moment to impose order on the chaos and create groupings of your own desire. Resource Groups are convenient for moving many Services from one Subscription (payment method) to another or to delete several Services.

## Service Plan - How much do you want to pay?

Or, how big of a machine do you want? This is where you decide how much compute power, storage space, memory, and bandwidth you're willing to pay for. Prices range from free for dev and test to much much more. You also need to choose where in the world the server will be located.

## Services - What do you need?

Now you can add Services to the machine you selected in the Service Plan or to a Resource Group (which one depends on the service). There are literally billions of services to choose from. Each one will add their own costs based on usage and daily rates to the credit card on the Subscription.
