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

~~While setting up a quick demo that uses the Supabase JS library for auth to show that access and refresh tokens are stored in localStorage, I stumbled across another issue: the Supabase auth API exposes the application to user enumeration attacks. That is, trying to sign up with an email that is already registered will return~~

```json
{
  "code": 400,
  "msg": "User already registered"
}
```

~~when it should ideally be indistinguishable to the end user. A good way to handle it would be to require a user to click a link sent to their email to confirm the account.~~

---

~~Why am I picking on Supabase so much? Even OWASP says "The decision to return a generic error message can be determined based on the criticality of the application and its data" [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#authentication-responses), and while the auth implementation with JWTs stored in localStorage isn't ideal, it's more important to focus on preventing XSS.~~

~~These aren't gaping security holes in my opinion, but here's why they are unacceptable: Supabase is a generic product designed to save developers the work of implementing backend code, and by having "good enough" security (with no apparent intention to improve it), they send the message to developers like me, "don't roll your own auth; don't even implement your own JSON APIs; pay us for a solution that's quick to set up, just don't look too closely because we don't follow OWASP best practices and have no plans to."~~

~~Last time I checked, the firebase JS SDK also stored access and refresh tokens in local Storage on web, but they may have changed this.~~

---

# Big Picture

Why does JWT auth or "stateless" auth even exist? As far as I can tell, it's to make horizontally scaling backend services easier. I'm not sure how feasible it is to use Cookie + session based auth with mobile or desktop apps, but... man that would be great.

---

# Conclusion

~~Is this actually an important security concern? According to OWASP, it is a concern, but in the hierarchy of importance I would put data sanitation and query parameterization to prevent SQL injection. The reason I believe it is worth thinking about this relatively minor security flaw is that it is architectural: SPA auth with JWTs provides 2 bad options: a simple implementation with numerous known security flaws, or a secure implementation that follows OWASP best practices but requires added complexity (e.g. BFF pattern).~~

- For something I'm not supposed to do myself, I spend an awful lot of energy on auth. Why is it such a nightmare?
- Browser and auth conventions have not caught up to SPAs, so bad practices abound.
- Personally, I favor batteries-included server-side rendered frameworks for web projects requiring auth because they have tested cookie/session based auth that conforms to best practices (as far as I can tell).
- It's been said before but once you actually implement all the mitigations to make JWTs secure, they're no longer "blazing fast", "stateless" or "easy to use". So, this has also been said before but, if at all possible, don't use JWTs.
- Can we start using CSRF_Tokens/AntiForgeryTokens and cookie/session-based auth with SPAs and JSON APIs?
