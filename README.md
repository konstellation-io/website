# website

Konstellation website generated using [Hugo](https://gohugo.io) and the [Docsy](https://www.docsy.dev/) template.

## Development

### Requirements

- [Install NodeJS 12.18.3+](https://nodejs.org/en/download/)

- [Install Hugo 0.74.3+ extended version](https://gohugo.io/getting-started/installing/)

### Download submodules and install dependencies

Download docsy theme submodules:

```
git submodule update --init --recursive
```

Install docsy theme dependencies

```
npm install
```

### Start local server

Start the server at [localhost:1313](http://localhost:1313/website/):

```
hugo server
```

## Deployment

When you push new changes to master branch, a github action creates the website and publishes it to the github-pages.
