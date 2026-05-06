[Skip to content](#start-of-content)
[Skip to sidebar](#sidebar)

/
[Blog](https://github.blog/)

* [Changelog](https://github.blog/changelog/)
* [Docs](https://docs.github.com/)
* [Customer stories](https://github.com/customer-stories)

[Try GitHub Copilot CLI](https://github.com/features/copilot/cli?utm_source=blog-top-nav-cli-features-cta&utm_medium=blog&utm_campaign=dev-pod-copilot-cli-2026)
[See what's new](https://github.com/events/universe/recap?utm_source=k2k-blog-tap-nav&utm_medium=blog&utm_campaign=universe25)

Search

* [Changelog](https://github.blog/changelog/)
* [Docs](https://docs.github.com/)
* [Customer stories](https://github.com/customer-stories)

[See what's new](https://github.com/events/universe/recap?utm_source=k2k-blog-tap-nav&utm_medium=blog&utm_campaign=universe25)
[Try GitHub Copilot CLI](https://github.com/features/copilot/cli?utm_source=blog-top-nav-cli-features-cta&utm_medium=blog&utm_campaign=dev-pod-copilot-cli-2026)

[Back to changelog](https://github.blog/changelog/)

Retired

January 29, 2026 •
1 minute read

# Closing down notice of legacy Copilot metrics APIs

![](https://github.blog/wp-content/themes/github-2021-child/assets/img/featured-v3-deprecations.svg)

## Table of Contents

* [What do these APIs do?](#what-do-these-apis-do)
* [Why?](#why)
* [What’s recommended moving forward?](#whats-recommended-moving-forward)

Menu. Currently selected: What do these APIs do?

* [What do these APIs do?](#what-do-these-apis-do)
* [Why?](#why)
* [What’s recommended moving forward?](#whats-recommended-moving-forward)

We are closing down three legacy Copilot metrics APIs as part of our transition to newer, more comprehensive usage metrics endpoints. Support for these APIs will be limited, and no new features will be developed.

* The User-level Feature Engagement Metrics API and Direct Data Access API will sunset on March 2nd, 2026.
* The Copilot Metrics API will sunset on April 2nd, 2026.

We strongly recommend transitioning any existing workflows to the [latest Copilot usage metrics endpoints](https://docs.github.com/enterprise-cloud%40latest/rest/copilot/copilot-usage-metrics?apiVersion=2022-11-28) to avoid disruption.

### [What do these APIs do?](#what-do-these-apis-do)

These APIs provide visibility into adoption and usage of GitHub Copilot features:

* [User-level Feature Engagement Metrics API](https://docs.github.com/enterprise-cloud%40latest/early-access/copilot/user-level-feature-engagement-metrics-api) offers a simplified view of user adoption of specific Copilot features through binary indicators.
* [Direct Data Access API](https://docs.github.com/enterprise-cloud%40latest/early-access/copilot/direct-data-access) provides user-level event data for Copilot code completion activity within supported IDEs.
* [Copilot Metrics API](https://docs.github.com/enterprise-cloud%40latest/rest/copilot/copilot-metrics?apiVersion=2022-11-28) provides aggregated usage metrics, primarily focused on IDE usage.

### [Why?](#why)

We have released a new set of Copilot usage metrics APIs that provide significantly more depth and flexibility. By consolidating to our latest APIs, we are able to:

* Provide a single, unified source of truth for Copilot metrics.
* Deliver more granular and valuable insights, including IDE agents, models, languages, lines of code, and fine-grained permissions.
* Reduce support overhead and streamline the product experience.

### [What’s recommended moving forward?](#whats-recommended-moving-forward)

We recommend transitioning to the [latest Copilot usage metrics APIs](https://docs.github.com/enterprise-cloud%40latest/rest/copilot/copilot-usage-metrics?apiVersion=2022-11-28), which provide comprehensive usage data at the enterprise, organization, and user levels.

These endpoints return aggregated metrics with expanded IDE telemetry, including:

* Edit modes and agent usage.
* Suggested versus accepted lines of code.
* Language and model breakdowns.
* Additional usage dimensions not available in the legacy APIs.

The new APIs are currently in public preview and will be generally available in the near future.

## Table of Contents

* [What do these APIs do?](#what-do-these-apis-do)
* [Why?](#why)
* [What’s recommended moving forward?](#whats-recommended-moving-forward)

Menu. Currently selected: What do these APIs do?

* [What do these APIs do?](#what-do-these-apis-do)
* [Why?](#why)
* [What’s recommended moving forward?](#whats-recommended-moving-forward)

[account management](https://github.blog/changelog/2026/?label=account-management)
[copilot](https://github.blog/changelog/2026/?label=copilot)
[enterprise management tools](https://github.blog/changelog/2026/?label=enterprise-management-tools)

Share
Copied
Shared

[Back to changelog](https://github.blog/changelog/)

## Related Posts

### May.01 Retired

[Upcoming deprecation of GPT-5.2 and GPT-5.2-Codex](https://github.blog/changelog/2026-05-01-upcoming-deprecation-of-gpt-5-2-and-gpt-5-2-codex)

[copilot](https://github.blog/changelog/2026/?label=copilot)

### Apr.30 Release

[GitHub Copilot in Visual Studio — April update](https://github.blog/changelog/2026-04-30-github-copilot-in-visual-studio-april-update)

[copilot](https://github.blog/changelog/2026/?label=copilot)

### Apr.27 Retired

[Copilot Student GPT-5.3-Codex removal from model picker](https://github.blog/changelog/2026-04-27-copilot-student-gpt-5-3-codex-removal-from-model-picker)

[copilot](https://github.blog/changelog/2026/?label=copilot)

### Apr.27 Improvement

[Copilot cloud agent starts 20% faster with Actions custom images](https://github.blog/changelog/2026-04-27-copilot-cloud-agent-starts-20-faster-with-actions-custom-images)

[actions](https://github.blog/changelog/2026/?label=actions)
[copilot](https://github.blog/changelog/2026/?label=copilot)

...
+1

### Apr.27 Release

[GitHub Copilot code review will start consuming GitHub Actions minutes on June 1, 2026](https://github.blog/changelog/2026-04-27-github-copilot-code-review-will-start-consuming-github-actions-minutes-on-june-1-2026)

[actions](https://github.blog/changelog/2026/?label=actions)
[copilot](https://github.blog/changelog/2026/?label=copilot)

...
+1

### Apr.24 Improvement

[Notice about upcoming new format for GitHub App installation tokens](https://github.blog/changelog/2026-04-24-notice-about-upcoming-new-format-for-github-app-installation-tokens)

[actions](https://github.blog/changelog/2026/?label=actions)
[application security](https://github.blog/changelog/2026/?label=application-security)
[client apps](https://github.blog/changelog/2026/?label=client-apps)
[copilot](https://github.blog/changelog/2026/?label=copilot)

...
+3

### Apr.24 Release

[GPT-5.5 is generally available for GitHub Copilot](https://github.blog/changelog/2026-04-24-gpt-5-5-is-generally-available-for-github-copilot)

[copilot](https://github.blog/changelog/2026/?label=copilot)

### Apr.24 Improvement

[Inline agent mode in preview and more in GitHub Copilot for JetBrains IDEs](https://github.blog/changelog/2026-04-24-inline-agent-mode-in-preview-and-more-in-github-copilot-for-jetbrains-ides)

[copilot](https://github.blog/changelog/2026/?label=copilot)

### Apr.23 Improvement

[Copilot Chat improvements for pull requests](https://github.blog/changelog/2026-04-23-copilot-chat-improvements-for-pull-requests)

[copilot](https://github.blog/changelog/2026/?label=copilot)

## Subscribe to our developer newsletter

Discover tips, technical guides, and best practices in our biweekly newsletter just for devs.

Enter your email\*

Subscribe

By submitting, I agree to let GitHub and its affiliates use my information for personalized communications, targeted advertising, and campaign effectiveness. See the [GitHub Privacy Statement](https://github.com/site/privacy) for more details.

[Back to top](#start-of-content)

## Site-wide Links

### Product

* [Features](https://github.com/features)
* [Security](https://github.com/security)
* [Enterprise](https://github.com/enterprise)
* [Customer Stories](https://github.com/customer-stories?type=enterprise)
* [Pricing](https://github.com/pricing)
* [Resources](https://resources.github.com/)

### Platform

* [Developer API](https://developer.github.com/)
* [Partners](https://partner.github.com/)
* [Atom](https://atom.io/)
* [Electron](https://www.electronjs.org/)
* [GitHub Desktop](https://desktop.github.com/)

### Support

* [Docs](https://docs.github.com/)
* [Community Forum](https://github.community/)
* [Training](https://services.github.com/)
* [Status](https://www.githubstatus.com/)
* [Contact](https://support.github.com/)

### Company

* [About](https://github.com/about)
* [Blog](https://github.blog/)
* [Careers](https://github.com/about/careers)
* [Press](https://github.com/about/press)
* [Shop](https://shop.github.com/)

* © 2026 GitHub, Inc.
* [Terms](https://docs.github.com/en/github/site-policy/github-terms-of-service)
* [Privacy](https://docs.github.com/en/github/site-policy/github-privacy-statement)
* Manage Cookies
* Do not share my personal information

* [LinkedIn icon

  GitHub on LinkedIn](https://www.linkedin.com/company/github)
* [Instagram icon

  GitHub on Instagram](https://www.instagram.com/github/)
* [YouTube icon

  GitHub on YouTube](https://www.youtube.com/github)
* [X icon

  GitHub on X](https://twitter.com/github)
* [TikTok icon

  GitHub on TikTok](https://www.tiktok.com/%40github)
* [Twitch icon

  GitHub on Twitch](https://www.twitch.tv/github)
* [GitHub icon
  GitHub’s organization on GitHub](https://github.com/github)