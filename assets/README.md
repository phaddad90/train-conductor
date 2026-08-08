# assets

`social-card.html` renders the GitHub social preview image.

To produce the PNG: open it in a browser at a 1280x640 viewport and screenshot,
or run

    npx playwright screenshot --viewport-size=1280,640 assets/social-card.html assets/social-preview.png

Then upload at Settings > General > Social preview. GitHub does not expose this
via the API, so it is a manual upload.
