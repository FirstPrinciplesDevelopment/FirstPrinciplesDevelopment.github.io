---
layout: post
title: "SPAuthentication"
date: 2023-02-22 17:00:00 -0500
categories:
tags: programming
---

As a new web developer, one is often admonished not to implement authentication oneself but rather to use a web framework with a robust and tested auth system, such as Django or Ruby on Rails. After reading about the complexities of authentication and the myriad attacks faced by web apps, I consider this advice sound and try to follow it. I don't assume, however, that a given project has a robust auth system. I'm not qualified to be a security auditor, but I try to understand how the tools I use work to prevent common classes of attacks, such as [cross site request forgery (CSRF/XSRF)](https://owasp.org/www-community/attacks/csrf), [cross site scripting (XSS)](https://owasp.org/www-community/attacks/xss/). The first web framework I learned, Django, had mitigations in place for many common vulnerabilities, such as anti-CSRF tokens that the framework issued and verified behind the scenes for every form submission, and I felt good about that. But then I learned about JSON APIs and SPAs and that's when the trouble started.

In fact, I was still using Django when I first encountered the issue with SPA authentication, but now I was using Django Rest Framework to build JSON API endpoints rather than rendering HTML on the server. The problem was authenticating from the javascript frontend I built, and keeping the user logged in. Because it was a simple learning project and it was mentioned in the DRF documentation, I used JWTs as bearer tokens and stored them in local storage. I didn't see a better way to handle it, but I stared to feel that something was wrong. Work took me in a different direction, namely server side rendering, which makes authentication relatively simple: I use what's built into [.NET Core](https://dotnet.microsoft.com/en-us/), [Django](https://www.djangoproject.com/), or [Phoenix](https://www.phoenixframework.org/). But my damned curiosity would push me again to grapple with SPAuthentication.

While reading [Hacker News](https://news.ycombinator.com/) I stumbled across [PostgREST](https://postgrest.org/en/stable/). Now I love relational databases, and I got very excited by the idea of writing more SQL and less boilerplate application code, especially serialization and ORM code. So I immediately tried to use PostgREST and found myself once again frustrated by authentication, but this time I wasn't happy with a toy solution that wasn't production ready; I wanted to do it right. I had read from trusted sources that storing JWTs in local storage on the web was not secure, but if the goal is to support a mobile, dekstop and web app with the same API, then Cookie based authentication isn't an option. Fortunately, a startup called [SupaBase](https://supabase.com/) was excited about PostgREST too, and they built their entire company on top of it. They had already implemented an auth system, and since it was open source, I figured I would either self-host SupaBase or just look at their code to see how they handled authentication. To my horror, I saw that their Javascript client stores access and refresh tokens in local storage. I searched the issues to see if anyone else had noticed this and raised an issue. A few people had, and the SupaBase team was not very understanding.

---

# Big Picture

Why does JWT auth or "stateless" auth even exist? As far as I can tell, it's to make horizontally scaling backend services easier. I'm not sure how feasible it is to use Cookie + session based auth with mobile or desktop apps, but... man that would be great.

---

# Conclusion

Is this actually an important security concern? According to OWASP, it is a concern, but in the hierarchy of importance I would put data sanitation and query parameterization to prevent SQL injection. The reason I believe it is worth thinking about this relatively minor security flaw is that it is architectural: SPA auth with JWTs provides 2 bad options: a simple implementation with numerous known security flaws, or a secure implementation that follows OWASP best practices but requires added complexity (e.g. BFF pattern).
