---
title: Restart compose with new env
tags:
  - dev
  - docker
  - docker-compose
  - devops
---

When service runs and you want to restart it but with changed envirnment veriables just use `docker compose up -d`, it will discover that veriables were changed and will recreate container with new.