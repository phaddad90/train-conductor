# assets

`social-preview.png` (1280x640) is the GitHub social preview and the README header: the card rendered
whenever the repo URL is pasted into X, LinkedIn, Slack, Discord or iMessage.
`social-preview-alt.png` is a brighter variant with a stronger grid, which holds
up better at very small sizes.

GitHub does not expose social preview via its API, so upload it manually at
Settings > General > Social preview.

`social-card.html` is a text-based fallback card. Render it with:

    npx playwright screenshot --viewport-size=1280,640 assets/social-card.html out.png
