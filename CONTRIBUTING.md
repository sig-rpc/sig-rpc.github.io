<!-- omit in toc -->
# Contributing to Reasonable Performance Computing SIG's [Website & Knowledge Base](https://sig-rpc.github.io/)

First off, thanks for taking the time to contribute! ❤️

All types of contributions are encouraged and valued. See the [Table of Contents](#table-of-contents) for different ways to help and details about how this project handles them. Please make sure to read the relevant section before making your contribution. It will make it a lot easier for us maintainers and smooth out the experience for all involved. The community looks forward to your contributions. 🎉

> And if you like what we're doing, but just don't have time to contribute, that's fine. There are other easy ways to support the project and show your appreciation, which we would also be very happy about:
> - Star the repository
> - Post about SIG-RPC on social media
> - Mention the website at local meetups and tell your friends/colleagues

<!-- omit in toc -->
## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [I Have a Question](#i-have-a-question)
  - [I Want To Contribute](#i-want-to-contribute)
  - [Suggesting Enhancements](#suggesting-enhancements)
  - [Your First Code Contribution](#your-first-code-contribution)
  - [Improving The Documentation](#improving-the-documentation)
- [Styleguides](#styleguides)
  - [Commit Messages](#commit-messages)
- [Join The Project Team](#join-the-project-team)


## Code of Conduct

As a SIG within the Society of RSE, SIG-RPC and its members and contributors are bound by the Society's [Code of Conduct][event_coc] to ensure that SIG-RPC remains a welcoming community. Elected officers of SIG-RPC are additionally bound by the Society’s [Trustee Code of Conduct][org_coc]. Minor issues may be reported to the officers of SIG-RPC via [sig-rpc-managers@society-rse.org][officers_email] as a first point of call, more serious/urgent grievances can be raised with [coc@society-rse.org][coc_email] as explained in the [Complaints and Grievances Policy][grievances_policy].

## I Have a Question

> If you want to ask a question, we assume that you have seen the [website](https://sig-rpc.github.io/) and understand it's goal.

Before you ask a question, it is best to search for existing [issues](https://github.com/sig-rpc/sig-rpc.github.io/issues) that might help you. In case you have found a suitable issue and still need clarification, you can write your question in this issue. It is also advisable to search the internet for answers first.

If you then still feel the need to ask a question and need clarification, we recommend the following:

- Create a [new issue](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose)
- Provide as much context as you can, this will help reduce the need for us to ask for further information.
- If relevant, include details such as your operating system or language/compiler version.

We will then endeavour to respond to you as soon as possible.

## I Want To Contribute

> ### Legal Notice <!-- omit in toc -->
> When contributing to this project, you must agree that you have authored 100% of the content, that you have the necessary rights to the content and that the content you contribute may be provided under the same licence as the existing content. We encourage the use of LLMs and other tools for improving readability of your drafts, however if a significant proportion of the content you are submitting is produced by generative AI this should be noted in your pull request.

### Suggesting Improvements

This section guides you through submitting a suggestion for improvements to the Reasonable Performance Computing SIG's Website & Knowledge-base, **including completely new guides or website features and minor improvements to existing content**. Following these guidelines will help maintainers and the community to understand your suggestion and find related suggestions.

<!-- omit in toc -->
#### Before Submitting an Idea

- Check the [website](https://sig-rpc.github.io/), carefully to ensure your idea isn't redundant.
- Perform a [search of existing issues](https://github.com/sig-rpc/sig-rpc.github.io/issues) to see if a similar suggestion has already been made. If it has, add a comment to the existing issue to show your support and add additional information, instead of opening a new one.
- If you're suggesting a big change (more significant than a new guide), please ensure your idea fits with the scope and aims of SIG-RPC. It's up to you to make a strong case to convince the SIG's officers of the merits of your proposal.

<!-- omit in toc -->
#### How Do I Make a Good Suggestion?

Suggested improvements are tracked as [GitHub issues](https://github.com/sig-rpc/sig-rpc.github.io/issues).

- Select and follow the **[most appropriate issue template](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose)**.
- Use a **clear and descriptive title** for the issue to identify your suggestion.
- Be **specific about the location** on the website that your improvement corresponds to.
- **Explain why this improvement would be useful** to a significant fraction of users.
- You may also want to point out **relevant websites/projects** that cover the suggested idea, which could serve as inspiration. 
- If you think a dynamic feature of the website is broken or could be improved:
  - **Describe the current behaviour** and **explain which behaviour you expected to see instead** and why.
  - **Include the web browser you are using**, and any other hardware/software details that may be relevant (e.g. is it caused by an ad-blocker?).




### Your First Guide Contribution

Guides live under the `_optimisations/` and `_profilers/` directories respectively, grouped by language. Each guide is written in the [Kramdown](https://kramdown.gettalong.org/documentation.html) superset of Markdown (`.md`) and should:

- Setup the YAML header:
  - `level`, The priority of the guide in search results. Make a judgement call, more common issues should have a level closer to 1.
  - `published`, By default this should be set to `true`.
  - `authors`, The name of the author/s that have contributed to the guide.
  - `name`, A short and general name which describes the optimisation or the profiler's name.
  - `language`, An array of languages that the guide refers to. *Be careful to ensure this matches the capitalisation of other guides for the language!*
  - `subcategory`, *(Optimisations only)* Ways to sub-categorise the optimisation, e.g. the library it corresponds to.
  - `tags`, *(Optimisations only)* An array of terms which may help this guide be found when searching (with the website's crude JS search).
  - `style`, *(Profilers only)* The profiling styles the profiler supports (e.g., Function-Level, Line-Level, Memory, Timeline, Hardware-Metrics, Benchmarking)
  - `website`, *(Profilers only)* URL of the profiler's main website.
- Each guide should begin with a short paragraph to introduce the guide
  - This will also appear in the directory, before users click through to the guide
  - The first paragraph should be followed by `<!--more-->`, which tells Jekyll where to place the fold.
- Within the guide provide concise examples, code snippets, screenshots and references where applicable.
  - Optimisation guides can really benefit from a small usable example that readers can test and modify.
- Use headings (`##`, `###`, etc.) to organise sections.
- Keep the tone educational and practical.

You can use existing guides as templates—for example, [`_optimisations/python/numba.md`](https://github.com/sig-rpc/sig-rpc.github.io/blob/master/_optimisations/python/numba.md) or [`_profilers/python/cprofile.md`](https://github.com/sig-rpc/sig-rpc.github.io/blob/master/_profilers/python/cprofile.md).

Once written, open a pull request against the `master` branch.

In your PR description, include:

* A short summary of your contribution (e.g. the language and topic)
* The motivation or context for your guide
* Any references, experiments, or related work you used

A maintainer or peer reviewer will provide feedback and help with merging.

Once merged, your contribution becomes part of the shared SIG-RPC knowledge base. 🎉
Feel free to share your guide with the community or link it in future discussions.


### Improving an Existing Guide

If you've identified something that you would like to improve in an existing guide, feel free to make the changes and submit a pull request against `master`.

If the changes are significant, or are potentially controversial, consider instead creating an [issue](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?template=improve_page.yml) to get feedback on your idea before committing additional effort.

## Styleguide

### Formatting a Guide

*These style guidelines should be considered advisory, existing articles have not had them retro-actively applied.*

- It's preferred that guides are written in 3rd person, referring to `a user` rather than `you`, the latter could be considered accusatory by some users.
  - e.g. "A user can configure..." instead of "You can configure...".
- If a link within a guide is tangential, it should be marked with `{:target="_blank"}`, so that it opens in a new tab.
  - e.g. `[The R Inferno](https://www.burns-stat.com/documents/books/the-r-inferno/){:target="_blank"}`
- If referring to a specific function, it should be named alongside `()` e.g. `print()` to clearly denote that it is a function.
  - If more broadly describing a family of functions, then this is not required.
  - Arguments may also be included if particularly relevant, e.g. `print(message, separator)` or `print(char*)`.
- If linking another page within the website, the path should be from the root of the website.
  - e.g. `/optimisation/r/dont-grow-dataframes`
- If writing a guide for an optimisation pattern, including a small toy benchmark is a great way to allow readers to experiment with the optimisation locally.
- If writing a guide for something visual, e.g. a profiler with a GUI, include screenshots. These are useful to readers unfamiliar with the tool being explained.
- Images for articles should be stored in `/assets/`.
  - If a guide has a single image, it may be stored in the root of this directory.
  - For guides with multiple images, please create a dedicated sub-directory to avoid filename conflicts.
- Included images can be captioned and labelled (so that they can be cross-referenced).
  - e.g. the below example would produce a numbered and captioned figure, with alt text which could be linked from elsewhere within the page with `#THE-LABEL`.
```jekyll
{% figure caption:"THE-CAPTION" label:THE-LABEL %}
![THE-ALT-TEXT](/assets/snakeviz-example.png)
{% endfigure %}
```
- Authors should be attributed via `authors` property of the YAML header, which is rendered as string. Names or preferred aliases may be used.
  - If someone contributed, but does not rise to the level of authorship they could be noted in the commit message and/or pull request.

### Commit Messages

Try to keep your commit messages clear and to the point.

## Join The SIG-RPC Team

If you'd like to get involved with SIG-RPC, beyond contributing to the website please join our [mailing-list](https://groups.google.com/a/society-rse.org/g/sig-rpc) and [SocRSE Slack channel](https://ukrse.slack.com/archives/C07LNNHKSJV) to keep abreast of announcements and upcoming opportunities. Each year an AGM is held to elect new officers to direct the special interest group, typically in September/October.

Minutes of past AGMs and meetings can be found in our [minutes repository](https://github.com/sig-rpc/minutes).

<!-- omit in toc -->
## Attribution
This guide is based on the [contributing.md](https://contributing.md/generator) generator!
