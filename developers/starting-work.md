---
layout: default
title: Starting Work
description: Where to begin
parent: /developers.html
---

## How to start working

We use the Shopify CLI to work on our tasks, which creates a private local theme that no one can see without the link which is very helpful for working on tasks as it allows freedom to work without having to worry about theme changes or it being seen before it’s ready for review.

All existing Github repositories should have a file in them called  shopify.theme.toml .This file contains a config that will look something like this:
```
[environments.dev]
store = “store-name”
```

This is what will allow you to connect to the store when you run the command shopify theme dev -e dev in the terminal. This command will provide you with the links you need to access your theme and all code changes will now be pushed to this theme on file save as long as the terminal continues running this command.