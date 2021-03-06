---
title: "Quality of Life Tips and Tricks - Burp Suite"
date: 2019-10-13T20:48:26-07:00
draft: false
toc: false
comments: true
twitterImage: .png # populate this
categories:
- burp
- quality of life
---

Quality of life patch/update in the context of videogames is a patch that
focuses on fixing bugs instead of introducing new content. New features in these
patches are not ground-breaking but rather making the game easier to play[^1].

I have been using these things to make my life easier. I am publishing them
gradually and will refine them into one final page similar to the
[cheatsheet]({{< relref "page/cheatsheet.markdown" >}} "cheatsheet"). This page
also pairs really well with [automation]({{< relref "categories/automation" >}} "automation").

Each section starts with step by step instructions. Some items have extra notes
and finally the `Why?` section has the justification/usecase.

# Table of Contents: <!-- omit in toc -->

- [Match and Replace for Test Username/Passwords](#match-and-replace-for-test-usernamepasswords)
    - [Why?](#why)
- [Disable Cached Responses](#disable-cached-responses)
    - [Why?](#why-1)
- [Filter OPTIONS](#filter-options)
    - [Why?](#why-2)
- [Burp Should Not Capture Corporate Credentials](#burp-should-not-capture-corporate-credentials)
    - [Why?](#why-3)
- [Use Firefox For Testing](#use-firefox-for-testing)
    - [Why?](#why-4)
- [Less Noise from Firefox in Burp](#less-noise-from-firefox-in-burp)
    - [Why?](#why-5)
- [Reduce the Size of Burp Projects Size for Long Term Storage](#reduce-the-size-of-burp-projects-size-for-long-term-storage)
    - [Why?](#why-6)
- [Rearrange Burp Repeater Request and Response Tabs](#rearrange-burp-repeater-request-and-response-tabs)
    - [Why?](#why-7)
- [Use a Default Burp Config](#use-a-default-burp-config)
    - [Why?](#why-8)

## Match and Replace for Test Username/Passwords

1. Capture the login request and identify the parameters.
    1. For example, a POST request with `user=hackerman&password=hunter2`.
2. Create a rule in `Proxy > Options > Match and Replace`.
    1. Type: Request header
    2. Match: `user=zzz0`
    3. Replace: `user=hackerman`
    4. Comment (optional): `admin account`
3. Create a second rule
    1. Type: Request header
    2. Match: `password=xxx0`
    3. Replace: `password=hunter2`
    4. Comment (optional): `password for admin`
4. On the mobile device or in the web browser, type `zzz0` and `xxx0` instead of
   username and password to login.

Notes:

1. If the user or pass include `%`, URL-encode it in the replace section to `%25`.
   E.g., `password=hunter2%` should be `password=hunter2%25`.
2. I use `zzz` and `xxx` because they are easy to type on mobile devices.
   1. For webapps use meaningful names like `admin1` and `admin1pw`.
3. Keep the numbers for `zzz` and `xxx` relative. E.g., the password for user
   `zzz0` should be `xxx0`.
4. This can be used for other long/complex inputs (e.g., certain payloads?). But
   the biggest time-saver for me is entering credentials.

### Why?
When testing a mobile application, I do not want to type complex passwords into
a mobile device over and over again. This way I can just enter `zzz0` and `xxx0`
to login.

The same trick works for webapps when I have multiple sets of credentials with
different roles. Instead of copy/pasting from a credential document, I can just
enter `admin1` and `admin1pw` to login as admin or `user1`:`user1pw` for user1.

## Disable Cached Responses

1. Enable the following built-in rules in `Proxy > Options > Match and Replace`.
    1. `If-Modified-Since`
    2. `If-None-Match`

### Why?
Sometimes I need to analyze a response but I see a 304 in Burp's history. Then I
have to use the search feature in Burp to find the first instance of the request
and see the content. With those headers removed, there are hopefully no more 304s.

## Filter OPTIONS

1. Add the following extension to Burp.
    1. https://github.com/parsiya/Parsia-Code/tree/master/burp-filter-options
2. In `Proxy > HTTP History` click on filter.
3. Remove the check beside `CSS` under `Filter by MIME Type`.
4. Every **new** OPTIONS request is now hidden.

### Why?
Preflight requests add a lot of noise to Burp's HTTP history. Currently, Burp
does not have a specific filter for them. This extension replaces the responses
to OPTIONS requests with `text/css`. Then I can filter them all by removing
`CSS` from Burp's history.

The extension's technical details:

* [Filter OPTIONS Burp Extension]({{< relref "post/2019-04-06-hiding-options/index.markdown" >}} "Filter OPTIONS Burp Extension")


## Burp Should Not Capture Corporate Credentials

1. Use a temporary Burp project/session.
2. Login to the application, enter the domain or corporate credentials like a
   normal login.
3. Identify which requests to which domains contain them.
4. Start the main project in Burp.
5. Add those domains to [Burp's SSL pass through][burp-ssl-pass-through] at
   `Proxy > Options > SSL Pass Through`.
6. Do the same for every request that is not related to the test but contains
   sensitive info (e.g., Okta).

### Why?
If I am testing an application that uses SSO, I have to enter corporate
credentials to login. Usually I am not testing the login portal but the app
behind it. Burp will store these credentials which not something I like. I keep
Burp projects forever (see [Reduce the Size of Burp Projects Size for Long Term
Storage](#reduce-the-size-of-burp-projects-size-for-long-term-storage)), I do
not want my corporate credentials stored in Burp. By adding these domains to SSL
pass through, Burp does not capture them.

## Use Firefox For Testing
Use Firefox (or a clone like [Pale Moon][palemoon]) with for testing with Burp.
Optionally, use [Firefox Developer Edition][ff-dev-edition] which has extra dev
tools and can co-exist with normal Firefox.

If installing Firefox dev edition:

1. Open Firefox dev edition.
2. `about:preferences`.
3. Check the box `Allow Firefox Developer Edition and Firefox to run at the same time`.

### Why?

1. Firefox has its own certificate store.
    1. Avoid installing Burp's CA in the operating system's certificate store.
2. Firefox has its own proxy settings.
    1. Do not have to redirect all other applications to Burp (e.g., Outlook, Chrome).
    2. This also reduces noise.

## Less Noise from Firefox in Burp

1. Do not install any addons in the testing browser.
    1. I am not using this browser for normal browsing.
    2. I do not want anything to be blocked by addons such as uBlock or Ghostery.
    3. Reduces the noise in Burp.
2. [Create a new Firefox profile][multiple-ff-profiles] and use `user.js` from the link below:
    1. https://bitbucket.org/mrbbking/quieter-firefox/src/master/
    2. Blog: [Towards a Quieter Firefox by Brian King - Blackhills Infosec][quieter-firefox].
3. Use these FoxyProxy rules to prevent random requests from reaching Burp, by [Liamosaur][liamosaur]:
    1. https://gist.github.com/liamosaur/a527d285b5394180c4bf3197dc7d8035
    2. Alternatively, add these to SSL pass through in
       [Burp Should Not Capture Corporate Credentials](#burp-should-not-capture-corporate-credentials).

### Why?
Fewer requests in Burp's HTTP history == good.

## Reduce the Size of Burp Projects Size for Long Term Storage

1. I Keep my Burp projects along with my notes.
2. Compress Burp projects (zip, 7z, etc.) (best bang for time):
    1. Old Burp save states were compressed, projects are not.
3. Remove out-of-scope from Burp project (risky because data is removed):
    1. Add all in-scope domains to Burp (most likely already done during the test).
    2. `Project > Save Copy` and check `Save in-scope items only`.
    3. This will remove all out-of-scope items from the new copy.
4. Remove large items from history and target (if I have time):
    1. Remove all Burp history filters.
    2. Sort by size.
    3. Remove all but one copy of large static resources like images and js
       files.
    4. There will be extra copies in the `Target` tab too.

### Why?
Keeping Burp projects have saved my hide more than I can count. Before storing
Burp projects for long-term or cloud storage, I want to reduce their size.

## Rearrange Burp Repeater Request and Response Tabs
1. `Repeater (menu, not the tab) > View`.
2. Choose Top/bottom or left/right split.

### Why?
When creating screenshots for reports, it’s better to have the tabs on top of
each other. Reports are usually in portrait mode. With the top/bottom split, we
can show both tabs and get a nicer image.

Make sure to move the border between the tabs and scroll the data in each to
show only relevant data. E.g., if you have a lot of text in response that is
not needed, only take a screenshot of the part that is important for the issue
you are presenting.

## Use a Default Burp Config

1. Open Burp, make any changes and set settings.
    1. The changes depend on your preferences.
2. Save the project config and the user config separately. They are JSON files.
3. Combine these two config files. Simply add the keys from one to the other.
4. When starting a new project, use that config.
5. Update this config regularly and store it somewhere (e.g, git repo).

### Why?
It saves time and I do not have to make the same changes for every project.
Some options will step be project-specific but most are not.

Some suggestions:

1. Disable interception at startup (biggest timesaver for me).
2. Add SSO domains (see [Burp Should Not Capture Corporate Credentials](#burp-should-not-capture-corporate-credentials))
   to SSL pass through.
3. Enable match and replace rules for the
   [Disable Cached Responses](#disable-cached-responses) section.
4. Add placeholders for match/replace rules
   (see [Match and Replace for Test Username/Passwords](#match-and-replace-for-test-usernamepasswords)).
5. Fonts, sizes and themes.

<!-- Links -->
[apex-113]: https://www.ea.com/games/apex-legends/news/performance-update-may-2019
[quieter-firefox]: https://www.blackhillsinfosec.com/towards-quieter-firefox/
[liamosaur]: https://github.com/liamosaur
[burp-ssl-pass-through]: https://portswigger.net/burp/documentation/desktop/tools/proxy/options#ssl-pass-through
[ff-dev-edition]: https://www.mozilla.org/en-US/firefox/developer/
[palemoon]: https://www.palemoon.org/
[multiple-ff-profiles]: https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Multiple_profiles

<!-- Footnotes -->
[^1]: For an example, see [Apex Legends Update 1.1.3][apex-113].
