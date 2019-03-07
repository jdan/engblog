title: "Windows High Contrast Mode"
published_on: March 21, 2019
author: Diedra Rater
team: Web Frontend
...

At Khan Academy, we've been working to make our web product more accessible to a variety of users. Part of this work has involved updating our styling to be able to work well with Windows High Contrast Mode. This required a little bit of learning via trial-and-error, so I wanted to share some of the things we discovered to make it easier for other developers working with accessibility.

## What is it and who uses it?
Windows High Contrast Mode is an accessibility settings feature in the Windows operating system. As the name suggests, it changes the visual appearance of everything displayed to have higher contrast. This includes applications that are native to the Windows platform as well as web apps.

Khan Academy's logged out homepage without Windows High Contrast Mode:

<img
    src="/images/windows-high-contrast-mode/lohp_default.png"
    alt="Sreenshot of Khan Academy's logged out homepage without Windows High Contrast Mode. There are light gray borders, light gray backgrounds, a dark blue button, and text of different colors."
/>

Khan Academy's logged out homepage with Windows High Contrast Mode:

<img
    src="/images/windows-high-contrast-mode/lohp_whc.png"
    alt="Sreenshot of Khan Academy's logged out homepage with Windows High Contrast Mode. All the borders are now black, the background colors are white (including buttons), and all of the text is either blue (if it's a link) or black (if it's not a link)."
/>

The high contrast version of the web may not be as pretty, but it is easier to see and understand. It's especially useful for users with blurry vision and users who experience severe color-blindness.

There are several different versions of Windows High Contrast Mode that can be used. **The examples in this post use High Contrast White.**

<img
    src="/images/windows-high-contrast-mode/settings.png"
    alt="Screenshot of the Windows High Contrast Mode settings options in Windows. There are four options: 'High Contrast /#1', 'High Contrast /#2', 'High Contrast Black', and 'High Contrast White'. 'High Contrast White' has been selected and the settings is turned on."
/>

## What does it do?
### Background colors and images
Background colors and background images are removed and replaced with white.

Both of these styles would be overridden:
```css
background-color: lawngreen; /* No color for you. */
background-image: url("cousin_fal.gif"); /* No image either. */
```

The opacity of background color styles will remain the same, though. The color will just be changed to white.

In this code example, the background is set to 50% black. In Windows High Contrast Mode, it will instead be 50% white.
```css
background-color: rgba(0, 0, 0, 0.5); /* White as the driven snow if the driven show were 50% opacity. */
```

### Border and outline colors
For border colors and outline colors, it actually goes the other way. No matter what color or opacity you set, they'll be set to 100% black in Windows High Contrast Mode.
```css
outline: 2px dotted lavenderblush; /* Black as your heart. */
border: 2px solid lemonchiffon; /* And my coffee. */
```

### `<img>` and `<svg>` elements
`<img>` and `<svg>` elements are immune and will remain intact.

```html
<img src="cousin_fall.gif" alt="Cousin Fal's at it again"> <!-- Impervious to Windows High Contrast Mode. -->
<svg> ... </svg> <!-- An svg of me being too lazy to write an svg example. 1,000,000 defense against Windows High Contrast Mode. -->
```

### Font colors
Font colors work the same as border and outline colors but with one more detail - the text element's background will be set to white. This is because the `<img>` and `<svg>` elements are unchanged. If you have text on top of an `<img>` or `<svg>` element, the text could become impossible to see if the image were dark when the text becomes black. By forcing the text to be black *and* have a white background, Windows High Contrast Mode ensures that the user will be able to read the text regardless of what `<img>` or `<svg>` you may have behind it.

In this example, the text will be black with a white background.
```css
color: someothercsscolornameyoudidntknowexisted;
```

### Comparison
Here's a section of the screenshots of the Khan Academy logged out homepage. Here we can see examples of almost all the ways Windows High Contrast Mode alters our custom styling:

1. The background color of the dark blue button and the light gray horizontal list of domains below the image are now white.
2. The borders on all three buttons are now black.
3. The image of the sky remains intact because it's been added as an `<svg>` element.
4. All the text on the page has returned to its default browser color (black for non-links and blue for links), and the text on top of the `<svg>` image has a white background.

<img
    src="/images/windows-high-contrast-mode/comparison.png"
    alt="Section of the Khan Academy logged out homepage shown in two versions side by side. On the left side is the default mode, and on the right side is the Windows High Contrast Mode. The section shown includes a button that reads 'Learners, start here'. In the default version, the button has a dark blue background, white text, and no visible outline. In the Windows High Contrast Mode version, the button has a white background, black text, and a black outline. Behind the button is an image of a sky, which is the same in both versions."
/>

## Improve web applications in Windows High Contrast Mode - practical examples from our work
### Background colors and images
Use HTML `<img>`s and `<svg>`s instead of `background-color` and `background-image` CSS **for important elements in the design**. Windows High Contrast Mode removes these characteristics of the page for a reason, so I don't want to encourage you to override that override altogether. But sometimes there are elements on the page that are visually necessary for users to understand the user interface and interact with it accordingly.

#### Replacing CSS `background-image` with `<img>` element
In the discussions feature on video pages, users can add in their posts a link to a specific point in the video. We visually communicate that the button jumps to a specific time in the video with a little blue play image to the left of the timestamp. This image was originally added using CSS's `background-image`. Of course, it was being removed in Windows High Contrast Mode and a big empty space was left instead.

To fix this, we removed the `background-image` from the CSS and added the image back with a `img` element so it would appear in both default and Windows High Contrast Modes.

##### Before
Screenshot:

<img
    src="/images/windows-high-contrast-mode/timestamp_before.png"
    alt="Screenshot of the discussion feature on a video page. There's a post that reads 'At', a blank space, and then '0:23, does the picture display bacteria?'"
/>

CSS:
```css
.videoPlayButton {
    background: url('/images/play.png') no-repeat left;
    padding-left: 18px;
    background-size: 14px 14px;

    /* Non-relevant styling has been removed for simplicity. */
}
```

JSX:
```jsx
// This is the same in both versions.
const timestamp = "3:14"; // Just for an example, to simplify this code.
const timestampAria = `${timestamp} Click to jump to ${timestamp} in video`;

return (
    <button class='videoPlayButton' aria-label={timestampAria}>
        {timestamp}
    </button>
);
```

##### After
Screenshot:

<img
    src="/images/windows-high-contrast-mode/timestamp_after.png"
    alt="Screenshot of the discussion feature on a video page. There's a post that reads 'At', a little blue play icon, and then '0:23, does the picture display bacteria?'"
/>

CSS:
```css
.videoPlayButton {
    /* Non-relevant styling has been removed for simplicity. */
}
.videoPlayIcon {
    height: 14px;
    width: 14px;
    margin-top: -4px;
    vertical-align: middle;
}
```

JSX:
```jsx
// This is the same in both versions.
const timestamp = "3:14"; // Just for an example, to simplify this code.
const timestampAria = `${timestamp} Click to jump to ${timestamp} in video`;

const playIcon = `<img src="/images/play.png" class="youTubePlayIcon" />`;

return (
    <button class='videoPlayButton' aria-label={timestampAria}>
        {playIcon} {timestamp}
    </button>
);
```

#### Replacing CSS `background-color` with `<svg>` element
Above the discussions feature on video pages, the video itself has a large play button on top. The actual play icon is an `<svg>` HTML element, but it's white, and the color for the area around it was added using `background-color` CSS. Of course, in Windows High Contrast Mode, the `background-color` CSS was removed and replaced with white, which left users with a white icon on a white background. Oops!

There are a couple of different ways to fix this, but, in this case, we chose to replace the element behind the icon with another `<svg>`, which would retain its color. (Another option would be to allow the color of the `<svg>` element to be overridden, which is illustrated in the next example.)

##### Before
Screenshot:

<img
    src="/images/windows-high-contrast-mode/playbutton_before.png"
    alt="Screenshot of a video page. On top of the video, there's a big white circle with a black outline."
/>

CSS:
```css
.playButton {
    background-color: #1865f2;
    width: 121px;
    height: 121px;

    /* Non-relevant styling has been removed for simplicity. */
}
```

JSX:
```jsx
// This is the same in both examples.
const playIcon = `M1.6,9.9C1.5,10,1.3,10,1.2,10C1.1,10,1,10,0.9,9.9C0.7,9.8,0.6,9.6,
    0.6,9.4V0.6c0-0.2,0.2-0.4,0.4-0.5C1.1,0,1.4,0,1.6,0.1l7.6,4.5c0.2,0.1,
    0.3,0.3,0.3,0.5c0,0.2-0.1,0.4-0.3,0.5L1.6,9.9z`;

const PLAY_BUTTON_DIAMETER = 44;

<button
    aria-label="Play video"
    class="playButton"
    onClick={this.handlePlayButtonClick}
>
    <svg
        height={PLAY_BUTTON_DIAMETER}
        width={PLAY_BUTTON_DIAMETER}
    >
        <path
            d={playIcon}
            fill={Color.white}
            transform="scale(4.375, 4.375) translate(9.75, 8.25)"
        />
    </svg>
</button>
```

##### After
Screenshot:

<img
    src="/images/windows-high-contrast-mode/playbutton_after.png"
    alt="Screenshot of a video page. On top of the video, there's a big white circle with a black outline and a blue play icon in the middle of the white circle."
/>

CSS:
```css
.playButton {
    /* Non-relevant styling has been removed for simplicity. */
}
```

JSX:
```jsx
// This is the same in both examples.
const playIcon = `M1.6,9.9C1.5,10,1.3,10,1.2,10C1.1,10,1,10,0.9,9.9C0.7,9.8,0.6,9.6,
    0.6,9.4V0.6c0-0.2,0.2-0.4,0.4-0.5C1.1,0,1.4,0,1.6,0.1l7.6,4.5c0.2,0.1,
    0.3,0.3,0.3,0.5c0,0.2-0.1,0.4-0.3,0.5L1.6,9.9z`;

const PLAY_BUTTON_DIAMETER = 121;
const PLAY_BUTTON_RADIUS = Math.floor(PLAY_BUTTON_DIAMETER / 2);

<button
    aria-label="Play video"
    class="playButton"
    onClick={this.handlePlayButtonClick}
>
    <svg
        height={PLAY_BUTTON_DIAMETER}
        width={PLAY_BUTTON_DIAMETER}
    >
        <circle
            cx={PLAY_BUTTON_RADIUS}
            cy={PLAY_BUTTON_RADIUS}
            r={PLAY_BUTTON_RADIUS}
            stroke="#fff"
            strokeWidth="1"
            fill="#1865f2"
        />
        <path d={playIcon} fill="#fff" />
    </svg>
</button>
```

#### Allowing the `<svg>` color to be overridden using `fillColor`
The colors of `<svg>`s will remain intact in Windows High Contrast Mode, but only if the colors are set directly on `fill`. If the `svg` image is white in your design, you should avoid setting that white color using `fill: yourColor`. Instead set `fill` to `currentColor` and set the `color` on a wrapper element. Then, in Windows High Contrast Mode, the `color` will be set to black and the `svg` will adjust accordingly.

Here's an example of one place where we fixed this - logo in the global header on Khan Academy.

##### Before
Screenshot:

<img
    src="/images/windows-high-contrast-mode/logo_before.png"
    alt="Screenshot of the global header at the top of the Khan Academy website. The header is white. In the center of the header is the Khan Academy logo icon, which is green. To the right of the logo icon is a blank space."
/>

CSS:
```css
.logoSvg {
    fill: #fff;
}
```

JSX:
```jsx
return (
    <a aria-label="Khan Academy" href="/">
        <svg class="logoSvg" aria-hidden={true}>...</svg>
    </a>
);
```

##### After
Screenshot:

<img
    src="/images/windows-high-contrast-mode/logo_after.png"
    alt="Screenshot of the global header at the top of the Khan Academy website. The header is white. In the center of the header is the Khan Academy logo icon, which is green. To the right of the logo icon is dark blue text that reads 'Khan Academy'."
/>

CSS:
```css
.logoLink {
    fill: #fff;
}
.logoSvg {
    fill: currentColor;
}
```

JSX:
```jsx
return (
    <a class="logoLink" aria-label="Khan Academy" href="/">
        <svg class="logoSvg" aria-hidden={true}>...</svg>
    </a>
);
```

#### Border and outline colors
Use invisible borders and outlines to visually define edges of elements that are created through different background colors laid against each other. Since borders and outlines are automatically set to 100% opacity black, you can make them 0% opacity in default mode so they will only appear in Windows High Contrast Mode. Borders can have rounded edges and be restricted to specific sides, but they increase the size of the element. To avoid affecting the design in default mode, if you're retrofitting an existing design, you might want to use an `outline`.

This is something we encountered with our sign up/login modals, which are visually defined based on their light background colors against the dark partially-transparent backdrop.

(Side-note: Here you can see an example of a partially-transparent background color being set to white but with the same opacity as in the original CSS.)

##### Before
Screenshot:

<img
    src="/images/windows-high-contrast-mode/modal_before.png"
    alt="Screenshot of a Khan Academy video page. The page is has a partially transparent white veil over it. In the center of the page is a big white rectangle. In the rectangle is a login form. The white rectangle has no outline."
/>

##### After
Screenshot:

<img
    src="/images/windows-high-contrast-mode/modal_after.png"
    alt="Screenshot of a Khan Academy video page. The page is has a partially transparent white veil over it. In the center of the page is a big white rectangle. In the rectangle is a login form. The white rectangle has a black outline."
/>

CSS:
```css
.modal {
    border: 1px solid rgba(0,0,0,0);
}
```