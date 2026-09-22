# One-click email webhooks

Two hosted pages that turn a link in an email into a logged action, with a friendly confirmation for the person who clicked:

- **[Unsubscribe](pages/unsubscribe.html):** one-click opt-out for a mail-merge campaign.
- **[Reading confirmation](pages/reading-confirmation.html):** a "Yes, I'm reading this" link for a newsletter, tracked by edition and region.

## The problem

**Unsubscribe (Aug 2026).** We ran an outreach campaign to dental practices through a Zapier mail merge, sent from a personal mailbox. The goal was a clear answer from every practice: fill in the survey, or say no. Saying no had to take one click, and every opt-out had to be logged automatically so that person never got another follow-up.

**Reading confirmation (Sep 2026).** For a newsletter sent from Kajabi, we wanted to know who was actually reading it, not just who opened it.

In both cases, a Zapier Catch Hook can log the click, but on its own it shows the person a bare JSON "success" response.

## How it works

```mermaid
flowchart LR
    A[Email link<br/>?email=recipient] --> B[Page on our website]
    B -- fetch, no-cors --> C[Zapier Catch Hook]
    C --> D[Click logged]
    B --> E[Friendly confirmation<br/>shown to the person]
```

1. Each email carries a personal link built with the platform's merge tag, for example `https://yoursite.com/reading-confirmation?edition=80&region=US&email={{ email }}` in Kajabi.
2. The page reads the parameters and fires the webhook in the background.
3. The person sees a confirmation card, or an error card with a fallback.

## Results

Across three email rounds, 34% of practices answered the survey and 6% opted out with one click, each one removed from follow-ups automatically.

## Lessons from building it

- **Merge tags work inside links.** Kajabi and Zapier mail merges substitute the tag anywhere in the HTML, including an `href`. In Kajabi, insert it in the HTML view, not with the Personalize button while the cursor sits in the link text.
- **Watch for "+" in addresses.** Email platforms don't encode it, and `URLSearchParams` reads a raw "+" as a space. The second page uses a small regex parser instead.
- **Extra parameters are free.** `edition` and `region` are plain text in the link, so tracking them needed no change in Kajabi, only in the page and the Zap's action step.
- **`no-cors` is enough.** The page can't read Zapier's response, but the request still lands, which is all a log needs.

## Use it

1. Create a Zap with a **Webhooks by Zapier > Catch Hook** trigger and copy its URL.
2. Paste the URL into `WEBHOOK_URL` in the page.
3. Host the page on your site (any page that accepts custom HTML).
4. Link to it from the email with your platform's merge tag for the recipient's email.
5. Map `email` (and `edition`, `region` if used) in the Zap's action step.
6. Test with your own address, ideally one with a "+", and check that the hit lands in Zapier.
