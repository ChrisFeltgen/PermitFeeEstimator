This tool allows users to estimate permit fees for Pompano Beach.

This tool supports the following divisions:
  1. Building
  2. Engineering
  3. Fire Prevention
  4. Landscape
  5. Zoning

This tool does not estimate Impact Fees or application fees at this time.

## Linking directly to a division tab

The tool supports deep links so a specific division opens by default, which
also works when the tool is embedded in an iframe:

- Hash on the tool's own URL: `PermitFeeEstimater.html#fire`
- Query string on the tool's own URL: `PermitFeeEstimater.html?division=fire`

Valid values: `building`, `engineering`, `fire`, `landscape`, `zoning` (a few
common aliases, e.g. `fire-prevention`, are also accepted). Switching tabs in
the UI updates the hash live, so the current view can always be copied and
shared.

Because cross-origin iframes can't read the parent page's URL, if the
embedding page (e.g. the Pompano Beach site) links to a division via its own
`?division=` query string, the tool will also pick that up from
`document.referrer` as a fallback — though this does not work for a `#hash`
on the parent page, since browsers never send the fragment in the referrer.
For guaranteed deep linking, set the iframe's `src` directly to one of the
URLs above, or send `postMessage({ division: "fire" }, "*")` to the iframe.

## Private Provider rates (Building tab)

Building permits (only) qualify for reduced fee rates when the applicant uses
a licensed Private Provider for inspections, or for plan review and
inspections, instead of the Building Department. On the Building tab, this is
exposed via a collapsed "Using a Private Provider?" disclosure below the
permit inputs — closed by default so it doesn't distract typical applicants,
but easy to find for those who need it. Selecting a Private Provider service
type swaps in the reduced rates (and adjusted minimums) for whichever
Building permit type is selected, and the result card notes which rate
schedule was applied.

## Iframe height sync

Since the tool is embedded via iframe on the Pompano Beach site and its
content height changes as users switch tabs, calculate fees, or expand the
Private Provider disclosure, it posts its height to the embedding page
whenever the content changes:

```js
{ source: "permit-fee-estimator", type: "resize", height: 1234 }
```

The embedding page has to opt in to act on it — it isn't automatic just by
embedding the iframe:

```js
window.addEventListener("message", (e) => {
  if (e.data && e.data.source === "permit-fee-estimator" && e.data.type === "resize") {
    iframe.style.height = e.data.height + "px";
  }
});
```

Height changes are detected with a `MutationObserver` on `<body>` (DOM
changes, not layout/box-size changes, since that's what actually drives this
page's height) and reported after a short debounce.

## Accessibility

- Each division's results panel is an `aria-live="polite"` region, so screen
  readers announce the fee estimate (or validation error) as soon as
  Calculate is pressed, without requiring focus to move.
- All numeric inputs use `inputmode="decimal"` so mobile browsers show a
  numeric keypad instead of the full keyboard.
