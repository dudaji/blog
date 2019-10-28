---
layout: post
title: "public access to gitlab private repo's file using gitlab pages"
author: yoyota
categories: general
comments: true
---

# How to
### Using gitlab pages access control
[gitlab page access control](https://docs.gitlab.com/ee/user/project/pages/introduction.html#gitlab-pages-access-control-core) allows public access to private repo's gitlab page

The Pages access control dropdown allows you to set who can view pages hosted with GitLab Pages, depending on your project’s visibility:

- If your project is private:
  - Only project members: Only project members will be able to browse the website.
  - Everyone: Everyone, both logged into and logged out of GitLab, will be able to browse the website, no matter their project membership.

> You can set gitlab page permission in  
> Settings > General > Pemissions 

# Example
Add .gitlab-ci.yml to your gitlab project

```yaml
pages:
  stage: deploy
  script:
    - mv test/test.txt public/test.txt
  artifacts:
    paths:
      - public
```

if your gitlab page url is:  
https://example.gitlab.io/example-group/example-project

access test
```
curl https://example.gitlab.io/example-group/example-project/test.txt
```
