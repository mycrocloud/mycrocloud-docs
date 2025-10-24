---
sidebar_position: 1
---

# Preparation

# Requirement
- GitHub Account
- Auth0 Account with AUTH0_DOMAIN

# Add Auth0's GitHub Social Connection
- Create new app at [GitHub Developer Settings: OAuth Apps](https://github.com/settings/developers) with this authorization callback url: `https://AUTH0_DOMAIN/login/callback`

- Add GitHub Social Connection with `Client ID` and `Client secret` from GitHub OAuth app
