# Mock-first AI Development

AI apps should run without API keys in local development.

Mock mode improves developer experience because it separates app behavior from provider availability, rate limits, quota, and cost.

## Benefits

- contributors can run tests without external accounts
- CI can execute without secrets
- onboarding is faster
- fixtures make regressions reproducible
- debugging is clearer because provider failures are isolated

## What to mock

- model responses
- tool outputs
- uploaded documents or images
- external API calls
- edge cases such as malformed outputs or timeouts

## Rule of thumb

If a developer cannot run the core app in five minutes without paid credentials, the sample is harder to learn from and harder to maintain.
