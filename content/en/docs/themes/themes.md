---
title: "Themes"
weight: 1000
toc: true
---

## General structure

```json
{
	id: "your-unique-identifier",

	name: "The Name of Your Theme",
	author: "your-name",
	desc: "Describe your fabulous theme",
	base: "light",

	props: {
    …
  }
}
```

The `id` must be unique across all the existing themes anywere. We
suggest using a version 4 UUID (you may have a `uuidgen` command, or
you may use [an online generator](https://www.uuidgenerator.net/))

Name, author, and description are pretty much free-form.

`base` must be either `light` or `dark`, and it marks your theme as a
light theme or a dark theme, plus it pulls in the default values from
the built-in base theme of that kind.

### List of properties

These are the keys you can set inside the `props` dictionary. A few of
them are probably not used, and many are used in several places. The
simplest way to see where a key is used is to set it to something
obviously out of place (e.g. `#FF0000` full red) and see what happens.

#### Colours

`accent` `accentDarken` `accentLighten` `accentedBg` `focus` `bg`
`acrylicBg` `fg` `fgTransparentWeak` `fgTransparent` `fgHighlighted`
`fgOnAccent` `fgOnWhite` `divider` `indicator` `panel`
`panelHighlight` `panelHeaderBg` `panelHeaderFg` `panelHeaderDivider`
`acrylicPanel` `windowHeader` `popup` `shadow` `header` `navBg`
`navFg` `navHoverFg` `navActive` `navIndicator` `link` `hashtag`
`mention` `mentionMe` `renote` `modalBg` `scrollbarHandle`
`scrollbarHandleHover` `dateLabelFg` `infoBg` `infoFg` `infoWarnBg`
`infoWarnFg` `switchBg` `buttonBg` `buttonHoverBg` `buttonGradateA`
`buttonGradateB` `switchOffBg` `switchOffFg` `switchOnBg` `switchOnFg`
`inputBorder` `inputBorderHover` `listItemHoverBg` `driveFolderBg`
`wallpaperOverlay` `badge` `messageBg` `success` `error` `warn`
`codeString` `codeNumber` `codeBoolean` `deckBg` `htmlThemeColor` `X2`
`X3` `X4` `X5` `X6` `X7` `X8` `X9` `X10` `X11` `X12` `X13` `X14` `X15`
`X16` `X17`

These can be specified in many ways:

* `#112233` RGB hex codes
* `rgb(0.5, 0.7, 1.0, 0.5)` RGB floating-point tuples (values between
  0 and 1)
* `rgba(0.5, 0.7, 1.0, 0.5)` RGBA floating-point tuples (values
  between 0 and 1), the last component is transparency
* `@fg` reference to another colour
* `$something` reference to a constant (see below)
* `:darken<0.5<@fg` a function call with one floating-point argument
  and one colour argument

The available functions are: `darken` `lighten` `alpha` `hue`
`saturate`. The colour argument can be in any of the above forms (so
yes `:darken<2<:alpha<0.4<@fg` is a valid colour specification: it
takes the `fg` colour, makes it more transparent, then makes it
darker).

#### Borders

`panelBorder` is the only property that's not used directly as a
colour. The default value is something like `" solid 1px
var(--divider)`. The value starts with a double-quote to tell the
theme engine "don't try parsing this as a colour" (notice that it
*doesn't end* with another quote!). `var(--divider)` here is CSS
syntax, with an effect pretty much the same as `@divider` in the
colour specifications described above.

#### Fonts

(This feature is currently only implemented by Sharkey)

`fontFaceSrc` can be set to a string (no starting quote! this is never
parsed as a colour) describing the source of a font. It can be
anything that [the `FontFace` constructor can understand as a
source](https://developer.mozilla.org/en-US/docs/Web/API/FontFace/FontFace#source):

* `local(Arial)` a reference to a locally-installed font
* `url(https://example.com/somefont.ttf)` a URL to a
  network-accessible font file
* `url(https://example.com/somefont.ttf) format('truetype')` a URL and
  a specific format
* `url(https://example.com/somefont.woff) format('woff'),
  url(https://example.com/somefont.ttf) format('truetype'),
  local(Arial)` a comma-separated list of any of the above (each
  source is tried in order, the first one that the browser understands
  and can fetch is used)

`fontFaceOpts` can be set to a dictionary, that will be passed
straight to [the `descriptors` argument to the `FontFace`
constructor](https://developer.mozilla.org/en-US/docs/Web/API/FontFace/FontFace#descriptors). You
very rarely need this.

#### Constants

You can also add extra values for colour specifications, using keys
starting with `$`. For example:

```json
props: {
  "$base": "#aa0044",
  "fg": ":darken<3<$base",
  "bg": ":lighten<4<$base",
  "accent": "$base"
}
```

The difference between `$foo` and `foo` is that the latter will be set
as a CSS variable, so a user's custom CSS could refer to it, while the
former won't escape the theme.

## Example

This is one of the built-in Misskey themes:

```json
{
	id: "504debaf-4912-6a4c-5059-1db08a76b737",

	name: "Mi Botanical Dark",
	author: "syuilo",

	base: "dark",

	props: {
		accent: "rgb(148, 179, 0)",
		bg: "rgb(37, 38, 36)",
		fg: "rgb(216, 212, 199)",
		fgHighlighted: "#fff",
		fgOnWhite: "@accent",
		divider: "rgba(255, 255, 255, 0.14)",
		panel: "rgb(47, 47, 44)",
		panelHeaderDivider: "rgba(0, 0, 0, 0)",
		header: ":alpha<0.7<@panel",
		navBg: "#363636",
		renote: "@accent",
		mention: "rgb(212, 153, 76)",
		mentionMe: "rgb(212, 210, 76)",
		hashtag: "#5bcbb0",
		link: "@accent",
	},
}
```

This is a minimal theme that just sets a font:

```json
{
  id: "0fb6f73a-02ea-4722-8394-4b02188b7cfa",
  name: "Prank,
  author: "dakkar",
  desc: "Sets a funny font",
  base: "light",
  props:{
    fontFaceSrc: "url(https://fonts.gstatic.com/s/rubikbubbles/v3/JIA1UVdwbHFJtwA7Us1BPFbRNTE.ttf) format('truetype')",
  }
}
```
