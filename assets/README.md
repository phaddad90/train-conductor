# assets

`social-preview.png` (1280x640) is the GitHub social preview and the README
header: a steam locomotive and three carriages drawn as a phosphor-green
engineering blueprint. Upload it at Settings > General > Social preview, which
is manual because GitHub does not expose that field via its API.

`social-preview-alt.png` is a brighter variant with a stronger grid, which holds
up better at very small sizes.

`social-card.html` overlays the repo name, tagline and topic chips on top of
`social-preview.png`, for a titled version of the card. Render it with:

    npx playwright screenshot --viewport-size=1280,640 assets/social-card.html assets/social-titled.png
