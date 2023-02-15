|  Component  | Coverage  |  Bugs  |  Maintainability Rating  |
| :---------: | :-----:   |  :---: |  :--------------------:  |
|  KDL Project Template  | [![coverage][coverage-badge]][coverage-link] | [![bugs][bugs-badge]][bugs-link] | [![mr][mr-badge]][mr-link] |

[coverage-badge]: https://sonarcloud.io/api/project_badges/measure?project=konstellation-io_website&metric=coverage
[coverage-link]: https://sonarcloud.io/api/project_badges/measure?project=konstellation-io_website&metric=coverage

[bugs-badge]: https://sonarcloud.io/api/project_badges/measure?project=konstellation-io_website&metric=bugs
[bugs-link]: https://sonarcloud.io/component_measures?id=konstellation-io_website&metric=Reliability

[mr-badge]: https://sonarcloud.io/api/project_badges/measure?project=konstellation-io_website&metric=sqale_rating
[mr-link]: https://sonarcloud.io/component_measures?id=konstellation-io_website&metric=Maintainability

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
