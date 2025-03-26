---
title: Browser-based animation in embedded systems 
date: 2025-02-01 10:00:00 +0100
categories: [embedded]
tags: [raspberry-pi, svg, javascript, riot-js]     # TAG names should always be lowercase
description:
  This demonstrator of a battery management system can be accessed both locally and via Internet.
  A Raspberry Pi 4 inside runs a Chromium browser showing an SVG-animated GUI.
  Remote viewers can interact with the same demonstrator remotely using their browser.
---
[Nubacell Ltd](https://nubacell.com/) is a British startup developing management systems for multi-cell
batteries. They developed a suitcase-like [demonstrator](https://nubacell.com/demonstrator) to showcase
their technology. It has a built-in LCD display connected to a Raspberry Pi 4 that shows the state
of the battery and its cells in a graphic, interactive form.

The same interactive demonstration can take place online for those who prefer not to travel. A Web
browser running on a PC or a smartphone connects to the Web server running on the Raspberry Pi.

To avoid developing two independent graphical interfaces, a local and a Web one, the Raspberry Pi in the
demonstrator runs a Web browser too. It's Chromium connecting to `localhost`. Please see how the interface
looks in the video below.

{%
include embed/video.html
src='/assets/element_changing_3_1.mp4'
title='Nubacell suitcase-like demonstrator'
autoplay=true
loop=true
muted=true
%}

Note the wave running down the active cells when the battery is discharging. Such animation is trivial
for a modern PC, but achieving a smooth-running animation and a decent frame rate in a Web browser
running on a Raspberry Pi required a careful choice of the Javascript framework. I used
[Riot.js](https://riot.js.org/) because of its minimal overhead and direct access to the native
browser APIs and events. I could control the version and all settings of the embedded browser,
Chromium, which is performance-critical. Those who want to access the interactive demo remotely
should install a modern browser supporting the SVG standard; there's no fallback provided.

Here is the definition of the `<nu-cell>` custom tag representing a battery cell in Riot.js.
The graphics is SVG exported from a graphics editor. The attributes of the corresponding class
and arbitrary Javascript expressions can be used within attribute values in square brackets:
`<rect width="{2 + 2}">`.

```xml
<nu-cell>

  <svg width="{Math.ceil((width+8) * scale)}" height="{Math.ceil((height+8) * scale)}"
       viewBox="0 0 {width+8} {height+8}">
    <g transform="translate(4,4)" visibility="{state.type==='missing' ? 'hidden' : 'visible'}">
      <path class="nu-cell_outline {state.state==='broken' ? 'nu-cell_broken' : ''}"
             d="M {width-10},27
                a 10,10 0 0 1 10,-10 
                v -17
                h -124
                v 17
                a 10,10 0 1 1 0,20
                v 250
                h 124
                v -250
                a 10,10 0 0 1 -10,-10 z"
      />
      <rect class="nu-cell_free - capacity - {state.type}" x="10" y="{height - full_capacity_pixels}"
            width="{width - 2 * 10}" height="{full_capacity_pixels}" />
      <rect class="nu-cell_charged" x="10" y="{height - full_capacity_pixels * state.charge_percent / 100}"
            width="{width - 2 * 10}" height="{full_capacity_pixels * state.charge_percent / 100}" />
      <image id="{animated_rect_uuid}" href="/static/css/gradient.svg" width="104" height="80" x="10" y="-20"
             visibility="{(state.is_charging || state.is_discharging) ? 'visible' : 'hidden'}"/>
    </g>
  </svg>

  <script>
      // Method definitions omitted
  </script>

</nu-cell>
```

Several types of events in the lifecycle of the battery and individual cells cause changes in cell animation.
When the battery starts charging, a wave starts running upwards over all active cells. When the battery starts
discharging, the same wave runs down. No wave runs when the battery is idle (neither charging nor discharging).
Additionally, the BCM often changes the state of individual cells (typically between *idle* and *active*).
The wave animation is shown on active cells only, which requires phase synchronisation between waves running
across different cells.

Here's a simple JavaScript snippet handling changes in the cell state. The SVG animation natively built
into Chromium is hardware-optimized on Raspberry Pi and very efficient. The brutal and "manual"
way of changing the DOM of the Web page by assigning to `innerHTML` is much easier to read than
programmatically controlling existing `<animateTransform>` elements, and proved to work fast in this context.

```javascript
  if (state.is_charging) {
    console.debug("nu-cell adding charging animation");
    rect.innerHTML = `
        <animateTransform attributeName="transform" type="translate"
            from="0 ${offset}" to="0 0"
            begin="indefinite" dur="${ms_till_multiple_of_5}ms"
            attributeType="XML" />
        <animateTransform attributeName="transform" type="translate"
            from="0 ${bottom_y - top_y}" to="0 0"
            begin="anim${ts}.end" dur="5s" repeatDur="indefinite"
            attributeType="XML" />
    `;
  } else if (state.is_discharging) {
    console.debug("nu-cell adding discharging animation");
    rect.innerHTML = `
        <animateTransform attributeName="transform" type="translate"
            from="0 ${bottom_y - top_y - offset}" to="0 ${bottom_y - top_y}"
            begin="indefinite" dur="${ms_till_multiple_of_5}ms"
            attributeType="XML" />
        <animateTransform attributeName="transform" type="translate"
            from="0 0" to="0 ${bottom_y - top_y}"
            begin="anim${ts}.end" dur="5s" repeatDur="indefinite"
            attributeType="XML" />
    `;
  } else {
    console.debug("nu-cell deleting all animation");
    // The cell is neither charging nor discharging
    rect.innerHTML = "";
  }
```

To summarise, browser-native HTML5/JavaScript programming optimised for embedded systems and making the best use
of their limited resources can be very practical.

