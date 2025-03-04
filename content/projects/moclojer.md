---
title: "moclojer"
date: 2023-12-03T12:00:00+10:00
featured: true
weight: 1
description: "Simple and efficient HTTP mock server with specification in yaml, edn or OpenAPI"
github_url: "https://github.com/moclojer/moclojer"
docs_url: "https://docs.moclojer.com"
---

Simple and efficient HTTP mock server with specification written in `yaml`, `edn` or `OpenAPI`.

## Features

- **No-code API Mock**: Create mock APIs without writing code
- **Multiple Specification Formats**: Support for YAML, EDN, and OpenAPI 3
- **Dynamic Responses**: Generate dynamic responses based on request parameters
- **Multi-domain Support**: Host multiple mock APIs on different domains
- **External Body Support**: Load response bodies from external files
- **Open Source**: MIT licensed and community-driven

## Getting Started

### Docker

```bash
docker run -it \
  -p 8000:8000 -v $(pwd)/moclojer.yml:/app/moclojer.yml \
  ghcr.io/moclojer/moclojer:latest
```

### Manual Installation

```bash
bash < <(curl -s https://raw.githubusercontent.com/moclojer/moclojer/main/install.sh)
```

## Example

```yaml
# This mock register route: GET /hello/:username
- endpoint:
    # Note: the method could be omitted because GET is the default
    method: GET
    path: /hello/:username
    response:
      # Note: the status could be omitted because 200 is the default
      status: 200
      headers:
        Content-Type: application/json
      # Note: the body will receive the value passed in the url using the
      # :username placeholder
      body: >
        {
          "hello": "{{path-params.username}}!"
        }
```

[View on GitHub](https://github.com/moclojer/moclojer) | [Documentation](https://docs.moclojer.com)
