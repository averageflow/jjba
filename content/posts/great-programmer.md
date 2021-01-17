---
title: "Go File Descriptors"
date: 2020-11-13T23:00:00Z
weight: 1
aliases: ["/go-file-descriptors"]
tags: ["Go", "File Descriptors", "UNIX"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1605355422/jjba-site/blog/go-file-descriptors/file-descriptor_azndem.jpg"
alt: "Go"
caption: ""
relative: false
hidden: false
comments: false
description: "Wondering what to do with the dreaded too many open files error in your Go application?"
disableHLJS: false
---

+++
authors = ["Josep Jesus Bigorra Algaba"]
date = 2020-09-18T22:00:00Z
excerpt = "If writing code were a science, all developers would pretty much be the same. But it is not. And just like in art, no two developers have the same thinking or perception..."
hero = "https://res.cloudinary.com/dehs6irlh/image/upload/v1606135201/jjba-site/blog/great-programmer/great-programmer-highlight_xro9oz.jpg"
timeToRead = 5
title = "What makes a great programmer ?"

+++
If writing code were a science, all developers would pretty much be the same. But it is not. And just like in art, no two developers have the same thinking or perception while working towards the same outcome.

While some struggle to produce the desired outcome, to a few, it comes almost naturally, as if an epiphany hits them at the moment they start writing code or solve a problem.

![The holy fuel that drives any respectable software developer. Say no to artificial energy drinks.](https://res.cloudinary.com/dehs6irlh/image/upload/v1600515371/jjba-site/blog/great-programmer/coffee_vsepjh.jpg "The holy fuel that drives any respectable software developer. Say no to artificial energy drinks.")

And in a blog post, Steve McConnell, one of the experts in software engineering talks about an original study which was carried in the late 1960s by Sackman, Erikson, and Grant. They found that the ratio of initial coding time between the best and worst programmers was about 20 to 1.

And the most interesting thing was that they found no relationship between a programmer’s experience and code quality or productivity. In simple words, writing good code is not the only factor that differentiates a good programmer from a great one.

Let us start with the good programmers first.

## Who is a good programmer?

I would say it is someone with:

* Excellent technical skills and write clean, neat code.
* Solid knowledge of development techniques and problem-solving expertise.
* Understanding programming best practices and when to employ them.
* An abiding passion for programming and strive to contribute to the team.
* Respectable and likeable by other members of the team.

So if you are a programmer and you have all the above traits, Congratulations! You are a good programmer. Be proud of it.

## Now coming to the great ones.

* They are rare.
* Their productivity is 3 times that of a good programmer and 10 times that of a bad programmer.
* They belong to the top 1% who don’t just write code but have a set of intangible traits that keep them poles apart from other programmers.

## TLDR;

> Great programmer === Good programmer + a set of intangible traits

While it’s not easy, if you’re dedicated enough, here are those intangible traits which you can cultivate within yourself to transition from being a good programmer to becoming a great programmer:

## Abrupt learning capability.

They are sharp-minded and that means they have the ability to learn new technologies and aren't browbeaten by new technologies. They have the ability to integrate seemingly disparate bits of information and process information on the fly.

Every programmer will surely experience a situation where he or she doesn't know the answer. Great programmers will find different resources, talk to the right people and find the solution no matter how impossible it appears. The best skill anyone can possess is knowing how to learn, and great programmers have mastered the skill of self-learning.

A great programmer doesn't let his ego come in between his work and his learning process. If he needs to know something, he will approach anyone in the hierarchy; from the lowest to the highest.

## They balance pragmatism and perfectionism.

John Allspaw, Chief Technology Officer at Etsy makes a good point in his post [_On being a senior engineer_](https://www.kitchensoap.com/2012/10/25/on-being-a-senior-engineer/).

He says that top-notch developers are healthy skeptics, which tend to ask themselves and their peers’ questions while they work, such as:

> What could I be missing?
>
> How will this not work?
>
> Will you please shoot as many holes as possible into my thinking on this?
>
> Even if it’s technically sound, is it understandable enough for the rest of the organization to operate, troubleshoot, and extend it?

The idea behind these questions is that they perfectly understand the importance of peer review and by a solid peer-review only, good design decisions can be made. So they “beg” for the bad news.

A great programmer will tend to not trust their own code until they’ve tested it extensively. Having said that, they also have the ability to understand market dynamics and the need to ship the product at the earliest. So they have the ability to make both quick and dirty hacks and elegant and refined solutions, and the wisdom to choose which is appropriate for a given situation at hand.

Some lesser programmers will lack the extreme attention to detail necessary for some problems. Others are stuck in perfectionist mode. Great programmers balance the two with perfect precision.

## They have great intuition.

In the sixth book of _The Nicomachean Ethics_, the famous philosopher and statesman _Aristotle_discusses the fourth of five capabilities people need to have for attaining true knowledge and thus becoming successful in whatever they do: intuition.

_Aristotle_’s point is simple. Intuition is the way we start knowing everything and knowledge gained by intuition must anchor all other knowledge. In fact, this way of gaining knowledge is so foundational that justification is impossible. That’s because knowledge by intuition is not based on a series of facts or a line of reasoning to a conclusion.

Instead, we know intuitional truth simply by the process of introspection and immediate awareness. From Steve Jobs to Richard Branson to Warren Buffet, the intuitives are generally successful in whatever they do, because they can see things more clearly and find the best solutions to problems more quickly than others. No doubt, all these individuals have a huge storage of expert knowledge and experience.

But they also seem to have an abundance of intuition that comes naturally to them and which enables them to grasp the essence of complicated problems and find uncannily right solutions. Great programmers typically display an intuitive understanding of algorithms, technologies, and software architecture based on their extensive experience and good development sense. They have the ability to understand at a glance what tools in their arsenal best fit the problem at hand.

And their intuitive abilities extend well beyond development and coding. This makes them highly versatile in articulating both technical and non-technical problems with both a layman and a specialist audience.

They are visionaries and they love challenges and will often seek to break their own code (before others do) in their pursuit of excellence.

## They are master communicators.

To get your ideas across, you need to make it simple and communicate as unambiguously as possible. Sounds simple? Isn’t it? Damien Filiatrault has rightly said:

> Good communication skills directly correlate with good development skills.

But unfortunately, this lack of clarity is the root cause of all troubles at work. And this is because of a phenomenon called the Curse of Knowledge. In 1990, a Stanford University graduate student in psychology named Elizabeth Newton illustrated the curse of knowledge by studying a simple game in which she assigned people to one of two roles: “tapper” or “listener.”

Each tapper was asked to pick a well-known song, such as “Happy Birthday” and tap out the rhythm on a table. The listener’s job was to guess the song. Over the course of Newton’s experiment, 120 songs were tapped out. Listeners guessed only three of the songs correctly: a success ratio of 2.5%.

But before they guessed, Newton asked the tappers to predict the probability that listeners would guess correctly. They predicted 50%. The tappers got their message across one time in 40, but they thought they would get it across one time in 2.

![A software developer in his natural habitat, hidden behind his favourite IDE and using a dark editor color scheme.](https://res.cloudinary.com/dehs6irlh/image/upload/v1600515371/jjba-site/blog/great-programmer/software-developer_nevets.jpg "A software developer in his natural habitat, hidden behind his favourite IDE and using a dark editor color scheme.")

Why did this happen? When a tapper taps, it is impossible for her to avoid hearing the tune playing along to her taps. Meanwhile, all the listener can hear is a kind of bizarre Morse code. Yet the tappers were flabbergasted by how hard the listeners had to work to pick up the tune. The problem is that once we know something — say, the melody of a song — we find it hard to imagine not knowing it.

Our knowledge has “cursed” us. We have difficulty sharing it with others because we cannot readily re-create their state of mind. That is why great programmers always confirm after communicating the message to the team.

They also can understand problems clearly, break them down into hypotheses and propose solutions cohesively. They understand concepts quickly or ask the right questions to understand, and above all, they don’t need every small bit to be written down in a document.

So if you want to become a great programmer, you need to make sure there is effective communication between you and your team. This not only keeps you at a higher plane of commitment but also shows your superiors that you are genuinely interested and invested in delivering a quality product.

## Last thoughts

So as you can see here, to be the best-of-class in your field, you don’t need any fancy degrees or even money to invest. All you need is an attitude to learn, be insanely curious and an intuitive ability to connect things based on the knowledge gained by you over the years.

Also important is the need to cultivate a healthy positive attitude, ditching that ego and have a tolerance to take and act on feedback. Once you do all this, I promise you will achieve greatness. As Bob Marley stated:

> The greatness of a man is not in how much wealth he acquires, but in his integrity and his ability to affect those around him positively.