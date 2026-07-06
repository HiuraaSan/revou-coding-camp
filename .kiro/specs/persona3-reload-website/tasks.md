# Implementation Plan: Persona 3 Reload Website

## Overview

Build a static multi-page website in HTML5, CSS3, and vanilla JavaScript that replicates the Persona 3 Reload pause menu aesthetic. The implementation proceeds in layers: project scaffolding → shared styles → SoundManager module → Intro animation → individual pages → accessibility hardening → tests.

## Tasks

- [ ] 1. Project scaffolding and asset setup
  - Create the full directory structure: `css/`, `css/pages/`, `js/`, `assets/audio/`, `assets/images/`, `fonts/`
  - Add placeholder audio files (`assets/audio/hover.mp3`, `assets/audio/click.mp3`) — short silent clips ≤ 500ms
  - Add self-hosted font files for Rajdhani or Orbitron under `fonts/`, or declare `@import` from Google Fonts as a fallback
  - Create `jest.config.js` with `testEnvironment: 'jsdom'` and a `tests/setup.js` bootstrap file
  - Add `package.json` with dev dependencies: `jest`, `jest-environment-jsdom`, `fast-check`
  - _Requirements: 6.1, 4.3, 5.3_

- [ ] 2. Global Pause Menu Theme stylesheet (`css/theme.css`)
  - [ ] 2.1 Define CSS custom properties (design tokens)
    - Declare `--color-bg: #0a0e1a`, `--color-accent: #00d4ff`, `--color-text: #e8f0ff`, `--color-panel-border`, `--color-active`, `--font-heading`, `--font-body`, `--transition-fast` on `:root`
    - Apply `background-color: var(--color-bg)` and `color: var(--color-text)` to `body`
    - _Requirements: 1.1, 1.5_

  - [ ] 2.2 Implement geometric angular decorations
    - Create reusable `.panel` and `.panel--angular` classes using `clip-path: polygon(...)` for trapezoid/diagonal-cut shapes
    - Implement `::before`/`::after` pseudo-element corner cuts for UI panels
    - _Requirements: 1.2, 1.6_

  - [ ] 2.3 Add glow accent and typography styles
    - Apply `font-family: var(--font-heading)` to all headings and `.nav-label` elements
    - Create `.accent-line` utility class with `box-shadow: 0 0 8px var(--color-accent)` and a cyan `border` or `outline`
    - _Requirements: 1.3, 1.4_

  - [ ] 2.4 Wrap all animations and transitions with `prefers-reduced-motion`
    - Move all `@keyframes` declarations and `transition` rules inside `@media (prefers-reduced-motion: no-preference)` blocks
    - Add an override block `@media (prefers-reduced-motion: reduce)` that sets `animation: none` and `transition: none` for the same selectors
    - _Requirements: 7.5_

  - [ ]* 2.5 Write property test for reduced-motion CSS (Property 13)
    - **Property 13: Reduced motion respected for all animated elements**
    - Parse `theme.css` and `intro.css`; for every rule containing `transition` or `animation`, assert an equivalent override exists in a `prefers-reduced-motion: reduce` block
    - **Validates: Requirements 7.5**

- [ ] 3. Nav component (`css/nav.css` + `js/nav.js`)
  - [ ] 3.1 Create nav HTML partial and nav stylesheet
    - Write the shared `<nav class="p3-nav">` markup with `<ul role="list">`, four `<a>` links, and the `#muteToggle` button
    - Style the nav with hover `:hover` background/border-glow using `box-shadow: 0 0 8px var(--color-accent)`, active `.active` state, and the angular clip-path decoration
    - _Requirements: 1.7, 2.2, 2.5, 6.6_

  - [ ] 3.2 Implement `nav.js` active-state and event wiring
    - On `DOMContentLoaded`, derive the active page from `window.location.pathname` and add `.active` to the matching `<a>`
    - Attach `mouseenter` listeners to all nav `<a>` and `<button>` elements to call `SoundManager.play('hover')`
    - Attach `click` listeners that call `event.preventDefault()`, invoke `SoundManager.play('click')`, then assign `window.location.href` after a short delay (~150ms)
    - _Requirements: 2.3, 2.4, 4.1, 5.1_

  - [ ]* 3.3 Write property test: nav present and structurally valid on every page (Property 2)
    - **Property 2: Nav component present and structurally valid on every page**
    - Load each page HTML file into jsdom; assert `<nav>` exists with ≥ 4 `<a>` children whose `href` values resolve to existing files
    - **Validates: Requirements 2.2, 2.5**

  - [ ]* 3.4 Write property test: active page highlighted (Property 3)
    - **Property 3: Active page is highlighted in the nav**
    - For each page, run `nav.js` with jsdom; assert exactly one `.active` class on the `<a>` matching the current page's path
    - **Validates: Requirements 2.4**

  - [ ]* 3.5 Write property test: nav link hrefs correct (Property 4)
    - **Property 4: Nav link hrefs are correct**
    - Iterate over all nav `<a>` elements on all pages; assert each `href` points to a file that exists in the project directory
    - **Validates: Requirements 2.3**

  - [ ]* 3.6 Write property test: mute control present on every page (Property 11)
    - **Property 11: Mute toggle control present on every page**
    - Load each page HTML into jsdom; assert exactly one `#muteToggle` element with a non-empty `aria-label`
    - **Validates: Requirements 6.6**

- [ ] 4. Checkpoint — Ensure all tests written so far pass
  - Run `npx jest --testPathPattern="nav|theme"` and verify all tests pass; resolve any failures before continuing.

- [ ] 5. SoundManager module (`js/sound-manager.js`)
  - [ ] 5.1 Implement `SoundManager` singleton with `init()`, `play()`, `mute()`, `unmute()`, and `isMuted` getter
    - Lazy-create `AudioContext` on first user interaction (`click` / `keydown` once-listener)
    - Fetch and decode audio files into `Map<string, AudioBuffer>` in `init()`; silence errors per the error-handling spec
    - `play(soundName)` creates a new `AudioBufferSourceNode`, connects to `destination`, calls `.start(0)`; if muted, exits early
    - `mute()` / `unmute()` set `_muted` and write `localStorage.setItem('p3-muted', ...)`; read the stored value in `init()`
    - Detect `typeof AudioContext === 'undefined'` and fall back to cloning an `HTMLAudioElement`
    - _Requirements: 4.1, 4.2, 4.5, 5.1, 5.2, 5.4, 5.5, 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 5.2 Wire the mute toggle button to SoundManager
    - In `nav.js` (or a dedicated block in `sound-manager.js`), attach a `click` listener to `#muteToggle` that calls `SoundManager.mute()` or `SoundManager.unmute()` and toggles `aria-pressed` + icon (`🔊` / `🔇`)
    - _Requirements: 6.3, 6.6_

  - [ ]* 5.3 Write property test: muted state suppresses all sound playback (Property 9)
    - **Property 9: Muted state suppresses all sound playback**
    - Use fast-check to generate arbitrary sound names; assert that calling `play(soundName)` while `isMuted === true` never invokes `AudioContext.createBufferSource` or `HTMLAudioElement.play`
    - **Validates: Requirements 6.4**

  - [ ]* 5.4 Write property test: mute preference round-trips through localStorage (Property 10)
    - **Property 10: Mute preference round-trips through localStorage**
    - Use fast-check to generate arbitrary mute/unmute toggle sequences; after each sequence, assert `localStorage.getItem('p3-muted')` equals the current `isMuted` state, and that re-calling `init()` restores that state
    - **Validates: Requirements 6.5**

  - [ ]* 5.5 Write property test: hover sound resets on rapid successive hovers (Property 8)
    - **Property 8: Hover sound resets on rapid successive hovers**
    - Spy on the AudioBufferSourceNode; when hover is playing, trigger a second `mouseenter`; assert the first source's `.stop()` was called and a new source was started
    - **Validates: Requirements 4.4**

  - [ ]* 5.6 Write unit test: audio file load failure is handled gracefully
    - Mock `fetch` to reject; call `SoundManager.init()`; assert no uncaught exception and subsequent `play('hover')` is a no-op
    - _Requirements: Error handling — audio load failure_

  - [ ]* 5.7 Write unit test: Web Audio API fallback path
    - Set `window.AudioContext = undefined`; call `SoundManager.init()` and then `play('hover')`; assert an `HTMLAudioElement.play()` call was made
    - _Requirements: 5.5_

  - [ ]* 5.8 Write unit test: autoplay deferral
    - Simulate a blocked `AudioContext` (constructor throws `NotAllowedError`); call `play('hover')`; assert it is queued in `_pendingPlays`; then dispatch a `click` event and assert the queued sound plays
    - _Requirements: 4.5_

- [ ] 6. Checkpoint — Ensure all SoundManager tests pass
  - Run `npx jest --testPathPattern="sound-manager"` and verify all tests pass; resolve any failures before continuing.

- [ ] 7. Nav hover / click sound integration (Property 1 and 7)
  - [ ] 7.1 Verify nav event listeners call SoundManager
    - Confirm `nav.js` correctly calls `SoundManager.play('hover')` on `mouseenter` and `SoundManager.play('click')` on `click` for all nav links and buttons
    - _Requirements: 1.7, 4.1, 5.1_

  - [ ]* 7.2 Write property test: nav hover triggers highlight and sound (Property 1)
    - **Property 1: Nav hover triggers visual highlight and sound**
    - Use fast-check to select arbitrary nav elements; dispatch `mouseenter`; assert the highlight CSS class is present and `SoundManager.play` was called with `'hover'` exactly once
    - **Validates: Requirements 1.7, 4.1**

  - [ ]* 7.3 Write property test: click sound plays before navigation (Property 7)
    - **Property 7: Click sound plays before navigation**
    - Use fast-check to select arbitrary nav links; spy on `SoundManager.play` and `window.location` assignment; dispatch `click`; assert `play('click')` is called before `location.href` changes
    - **Validates: Requirements 5.1**

- [ ] 8. Intro animation (`css/intro.css` + `js/intro.js` + HTML in `index.html`)
  - [ ] 8.1 Build the intro overlay HTML and CSS
    - Add `#intro-overlay` with `position: fixed; z-index: 9999; background: var(--color-bg)` to `index.html`
    - Include `.intro-logo` with title text and `#intro-skip` button (`tabindex="0"`, `aria-label="Skip intro"`)
    - Write `@keyframes` for logo fade-in (~1.5s), hold (~1s), and overlay fade-out (~1s), totaling ≤ 5s; wrap in `prefers-reduced-motion: no-preference`
    - _Requirements: 3.1, 3.4, 3.5, 7.6_

  - [ ] 8.2 Implement `intro.js` session logic
    - On `DOMContentLoaded`: check `sessionStorage.getItem('p3-intro-played')`; if set, remove overlay immediately
    - If not set, show overlay, start animations, attach skip button listener (click, Space, Enter)
    - `_endIntro()`: set `sessionStorage.setItem('p3-intro-played', '1')`, remove overlay, restore `document.body` scroll
    - Add 5000ms `setTimeout` fallback calling `_endIntro()` in case `animationend` does not fire
    - Wrap all `sessionStorage` calls in `try/catch` for graceful degradation
    - _Requirements: 3.1, 3.2, 3.3, 3.6, 3.7_

  - [ ]* 8.3 Write property test: intro shows first visit, skipped on repeat (Property 5)
    - **Property 5: Intro shows on first visit and is skipped on repeat (round-trip)**
    - Use fast-check to generate arbitrary session storage states (cleared vs. `"1"`); assert overlay is present when cleared and absent when flag is set
    - **Validates: Requirements 3.1, 3.2**

  - [ ]* 8.4 Write property test: intro completion and skip both set sessionStorage flag (Property 6)
    - **Property 6: Intro completion and skip both set the sessionStorage flag**
    - Invoke `_endIntro()` via both the timer path and the skip button path; assert `sessionStorage.getItem('p3-intro-played') === '1'` in both cases
    - **Validates: Requirements 3.3, 3.7**

  - [ ]* 8.5 Write unit test: intro duration does not exceed 5 seconds
    - Assert that the sum of all `@keyframes` durations declared in `intro.css` (or the `setTimeout` fallback value in `intro.js`) is ≤ 5000ms
    - _Requirements: 3.4_

- [ ] 9. Checkpoint — Ensure all intro tests pass
  - Run `npx jest --testPathPattern="intro"` and verify all tests pass; resolve any failures before continuing.

- [ ] 10. Home page (`index.html` + `css/pages/home.css`)
  - Add `<header>`, `<nav>` (shared markup), `<main>`, `<footer>` semantic structure
  - Implement hero section with circular clock motif decorative element (CSS `border-radius` + accent rings)
  - Link `theme.css`, `nav.css`, `intro.css`, `home.css`, `sound-manager.js`, `nav.js`, `intro.js`
  - _Requirements: 1.6, 2.1, 2.5, 7.1, 7.4_

- [ ] 11. About page (`about.html` + `css/pages/about.css`)
  - Add semantic structure identical to Home
  - Implement bio/project description inside an angular `.panel--angular` component
  - Include at least one decorative element (e.g., diagonal stripe dividers) using `::before`/`::after`
  - Link shared stylesheets and JS; omit `intro.js`
  - _Requirements: 1.2, 1.6, 2.1, 7.4_

- [ ] 12. Gallery page (`gallery.html` + `css/pages/gallery.css`)
  - Add semantic structure; implement image grid with `<figure>`/`<img alt="...">` elements
  - Apply diagonal stripe dividers between sections using CSS `linear-gradient` or `clip-path`
  - Ensure all `<img>` elements carry descriptive `alt` text
  - _Requirements: 1.6, 2.1, 7.2, 7.4_

- [ ] 13. Contact page (`contact.html` + `css/pages/contact.css`)
  - Add semantic structure; implement a contact form (`<form>`, labeled `<input>`, `<textarea>`, `<button type="submit">`) styled as a Persona 3 menu panel
  - Ensure form controls are keyboard-navigable with visible `:focus` indicators
  - _Requirements: 2.1, 7.3, 7.4_

- [ ] 14. 404 error page (`404.html`)
  - Build a standalone page with Pause_Menu_Theme styles; display an error message and a link back to Home
  - Include the shared nav markup; do not invoke intro or sound on 404
  - _Requirements: 2.6_

- [ ] 15. Keyboard accessibility and focus styles
  - Add visible `outline` or `box-shadow` `:focus-visible` styles to all `<a>`, `<button>`, `<input>`, `<textarea>` elements in `theme.css`
  - Verify `#intro-skip` is reachable via Tab and activatable via Enter/Space (confirm existing `tabindex="0"` and keyboard listener in `intro.js`)
  - Ensure all `<img>` elements across all pages have non-empty `alt` attributes
  - _Requirements: 7.2, 7.3, 7.6_

- [ ]* 15.1 Write property test: accessibility invariants hold for every page (Property 12)
  - **Property 12: Accessibility invariants hold for every page**
  - Load each page HTML into jsdom; assert: all `<img>` have non-empty `alt`; all interactive elements have `tabindex ≥ 0` and a `:focus` CSS rule; page contains `<header>`, `<nav>`, `<main>`, `<footer>`
  - **Validates: Requirements 7.2, 7.3, 7.4**

- [ ] 16. Final checkpoint — Full test suite and review
  - Run `npx jest --run` (or `npx jest`) and confirm all tests pass
  - Verify no broken internal links by opening each HTML file and inspecting nav hrefs
  - Confirm Lighthouse performance score ≥ 70 on the Home page (run Lighthouse CLI or DevTools manually)
  - Ensure all tests pass; ask the user if any questions arise before delivery.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Each task references specific requirements for traceability
- Checkpoints (tasks 4, 6, 9, 16) are natural integration gates — all tests should be green before moving past them
- Property tests use `fast-check` and run a minimum of 100 iterations each; every test file should include the tag comment `// Feature: persona3-reload-website, Property N: <property text>`
- Unit tests focus on error paths (audio load failure, Web Audio API fallback, autoplay deferral) that property generators do not cover
- All JS files run in a `jsdom` environment via Jest; no real browser is required for the test suite
- Lighthouse score verification (task 16) requires either a real browser or Lighthouse CI; it is not automated in Jest
