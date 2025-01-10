# Preconfigured local setup for bot development with Docker and Node.js

This template provides:

- Node.js with typescript, nodemon and a simple "Hello World" script
- MySQL container (the version of mysql can be changed)

## Installation

Checklist:

- [ ] clone / download this repo: https://github.com/help-me-mom/node-cli-docker-template
- [ ] if needed, update mysql version in [`compose.yml`](./compose.yml) in `services > database > image`  
  by default the latest one is used
- [ ] you can execute `docker compose up --build` to build and start containers  
  you might need to wait until containers finish their bootstrap: creating databases, etc.
- [ ] that's it, now you can commit changes to git and start development

## Coding

You can code in [`script/src/index.ts`](./script/src/index.ts).
