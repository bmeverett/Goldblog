---
title: "Helpful Tips for Accessibility"
date: "2020-01-03T12:34:56.117Z"
description: "Tips for turning both your engineering projects and overall organization into an accessibility machine."
link:
    title: JavaScript January
    url: https://www.javascriptjanuary.com/blog/helpful-tips-for-becoming-accessible
---

Hi there! 👋 I'm Josh Goldberg (Site/Twitter). I'm a frontend developer at Codecademy, a site where you can learn to code for free.

Over the last year we'e been making our site more accessible to users with different ways of using their computer, such as those with reading or visual impairments.

Some users experience subtle differences in computer usage, like dyslexia or a lot of screen glare. Others are partially or totally blind and rely on "screenreader" narration software to tell them what's on a webpage. Accomodating these users -virtually none of whom chose to rely on a website being accessible- is the morally correct thing to do, yet the vast majority of webpages have easily detectable failures. 😰

Accessibility is not a bolt-on. It's something that must be built into every product we make so that our products work for everyone. - Satya Nadella, CEO, Microsoft

If that doesn't convince you, well, you might be sued. 🙃

Preparing for Accessibility
The WCAG (Web Content Accessibility Guidelines) define various technical requirements for web pages. They are classified as A (casually: "most important"), AA (casually: "should have"), and AAA (casually: "nice to have"). You can read more on MDN or if you really enjoy deep technical specifications, w3.org. We at Codecademy are going for at least AA because we want a better-than-just-barely-working experience for our users, and AAA can sometimes be difficult to achieve.

If you're an organization interested in accessibility, the commitment is more than just a fixed engineering cost. Your whole product creation pipeline needs to buy in. Designers need to create designs that are behaviorally and visually accessible. PMs (product managers? program managers? whatever you happen to call them) must account for time spent fixing issues and running accessibility user testing. Everyone should agree on it being part of your goals and understand what level of accessibility you're striving for.

This post is a collection of the lessons we've learned and past mistakes we've corrected in the context of making our site accessible. Every example is something we've had to fix and re-fix repeatedly. I hope this can be useful to you in making your web app accessible!

Design Tips
I'm no designer, but I've seen enough accessibility bug reports and squinted at enough aXe complaints to see common trends in where well-intentioned designers create fundamentally inaccessible designs. Most importantly, design with accessibility in mind.

Build an accessible application
Color Contrast!
The single most common design accessibility violation I've seen by far is text not having enough color contrast against its backdrop. Too little contrast results in users with low vision, low quality monitors, or low light quality setups not being able to see the text clearly. It's as if it doesn't exist.

You can calculate the color contrast between two colors with the handy WebAIM Contrast Checker tool. Chrome and other dev tools have started adding in indicators too!

Do you see the piece of text in this image with too low color contrast?

The unit number "1" in the left of the image has only a 2.52:1 ratio, which is not enough for some users to read large text per AA requirements. 😢.

This improved contrast amount HAS A 5.8.1:1 ratio, which is enough for users to read large text per AA and even AAA requirements. 😍!

We should also note that some users are partially or fully color blind, so we can't rely on hues alone to distinguish between colors. Red and green for states are just subtly different shades of gray to them.

Buttons are Hard
Has this CSS ever happened to you?

```css
button:focus {
    outline: none;
}
```

By default, browsers show a "focus" outline around interactive elements such as buttons so that users can tell which element is active. Users who physically cannot use a mouse, as well as "power" users who opt not to in the interest of ⚡ speed, generally Tab and Enter around instead.

Quite a few sites manually remove the blue focus outline around their custom buttons because they don't like how it looks. But! Without a focus outline, users cannot visually tell which element is focused.

Can you tell which button is focused in this screenshot?

Trick question -- you can't. 😡

More insidious is the CSS style that manually adds in a different :focus indicator but does not adhere to web accessibility standards:

```css
button:focus {
    box-shadow: 0 0 3px 2px #ccc;
}
```

Some users neither use a mouse nor are able to see color contrast ratios below AA requirements. If your design team wants to define its own :focus styles, they should also adhere to color contrast requirements -- both against the background again between the initial vs. focused state. Low vision users need to be able to tell the difference between button states.

You can see the button styles for Codecademy's design system, Gamut, here. As of January 2020, we haven't finished making them all accessible. Can you spot which ones aren't yet usable against white and/or black backgrounds? 😉

Engineering Tips
Regardless of which blazingly fast frontend framework you use -if any-, there are prebuilt engineering helpers to stop you from making accessibility mistakes. These certainly won't catch all -or even a majority- of accessibility issues, but in my experience, roughly a third of accessibility issues I've dealt with were caught by them during builds.

Dynamic Analysis
Dynamic analysis is the art of scrutinizing what happens when you run your code: a.k.a. testing it! Hopefully you already have some sort of unit, integration, and/or end-to-end tests in your code in your build pipeline. aXe is a great industry standard addition to your end-to-end tests that can scan a site for obvious accessibility issues using its Cypress, Testcafe, or general axe-core library integrations.

You can also run the same aXe scans against your components in unit tests, such as with jest-axe. These will find much fewer and much more granular level issues, but can be much faster than end-to-end tests.

Get help building for accessibility
Static Analysis
Static analysis is the art of scrutinizing in your code without running it. You probably already run some form of it already, such as ESLint or Prettier(and if you don't yet, you should; they're very useful!). ESLint in particular is a great "linter", or tool that finds issues in your code statically. I prefer React and thus have access to the excellent eslint-plugin-jsx-a11y, which adds a host of accessibility checks as ESLint rules.

Image Alt Text
Images mean nothing to someone who can't see them. Non-sighted users (and search engine crawlers!) will not be able to understand an image in your webpage unless it's given some sort of descriptive label.

This kind of failure can be caught by both dynamic analysis and static analysis:

```html
<!-- What does this even mean!? -->
<img src="who-knows.svg" />
```

Purely decorative images such as backgrounds can receive alt="" to disable narration:

```html
<img alt="" src="background.svg" />
```

If the image is informative (relevant to the page content), such as an icon or informative photo, you can fill out the alt to have that narrated to users:

```html
<img alt="Delete" src="delete.svg" />
```

If your image already is labeled via existing text, use aria-labelledby to indicate that the two are linked:

```html
<img aria-labelledby="description" src="delete.svg" />
<span id="description">Delete</span>
```

Providing labels in HTML elements has the additional advantage of surfacing text to automatic translation services (demos+explanation).

Element Semantics
HTML elements have semantic meaning. A <p> tag indicates a paragraph of content. What does this element semantically mean to you?

```html
<div>Submit</div>
```

Is it a label? Is it a button? A title? 🤷‍

Now look at this one:

```html
<input type="submit">Submit</input>
```

👌 now that's a semantically understandable element.

Browsers work best when you use the right HTML elements for what they're meant to do. Attaching onClick listeners and the like to regular <div>s is bad practice because those elements don't receive Tab focus by default. Enter s only apply to focused elements.

Even if you make your non-interactive element focusable with tabIndex="0", you'll still have to add custom event listeners for Enter s. So please, for your own sake, use the right HTML elements for the job.

Static analysis can generally detect that you've added an interactive listener to a non-interactive element. Dynamic analysis generally can't. 😢

User Testing
Lastly, the only way to really know your accessibility status is to run it by real-life testers trained on detecting these kinds of issues. That could be your own company testers or a third-party user testing service.

For reference, we have a fancy custom agreement with test.io to run accessibility tests monthly. During bug prioritizion we treat the bugs as affecting 5% of our users. Your mileage may vary. The important part is to integrate accessibility testing into your process - otherwise it can easily be forgotten and ruined.

In Conclusion
Accessibility is a great thing to do and both morally and legally important. It's also a non-trivial investment in time, training, and resources.

Further Resources
Inclusive Components is an excellent engineering resource that I'd highly recommend to any frontend developer.

Feel free to tweet @ or DM me on Twitter -- though I'll probably redirect you to an area expert.

Best of luck! ♿🙌

