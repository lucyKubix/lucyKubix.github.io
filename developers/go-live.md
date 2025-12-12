---
layout: default
title: Go Live
description: GIG of a lifetime
parent: /developers.html
---

## Go live

Go lives can only happen between 8:30 and 15:00 Monday to Thursday. No go lives should be happening in the evenings or on Fridays except for in extremely rare cases such as any cart issues or anything that directly affects conversion and requires sign off from PM, client and Lead. In the time where the lead is unavailable, sign off is required from the Production lead, senior developer and developer who worked on the task. If a/the senior is the person who worked on the task, they can ask one of the other midweights. The reason on sign off is to way up the benefits vs risks of going live outside of the hours along with the certainty of the code.

In normal circumstances, the process for go lives is as follows:
Once the code has been approved by another member of the team through the pull request process and reviewed and approved by the PM and client it can go live.
Create a backup of the main theme on the relevant store. Note that this may not always be the live theme as the client may have pushed their own theme to live so make sure the theme you backup is the one connected to the main Github branch.
Rename the backup using the following naming convention: KBX BU [DATE] @[TIME], for example KBX BU 1st Oct @09:52
Once the backup has finished, create a pull request in Github merging your branch to main.
Once merged, delete the branch to keep the repository tidy, check over the site to make sure the changes are okay, and update the relevant ClickUp task informing the PM that the changes are live.
Keep the site open in an incognito window to test the live theme immediately after deployment.
If a rollback is needed, the backup you created can be published instantly — always keep it untouched until confirmed everything is stable.
If some customiser setup is needed, make sure to note that the client needs to set this up when responding on the ticket. Ensure AM/client has all they need to know that this live and/or what they need to do to see this on the live site
