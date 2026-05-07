---
author: Negar Baharmand
pubDatetime: 2026-05-07T09:00:00.000Z
title: "Building an Accessible Camera Flow in React: Permissions, Failure States, and Clean UX"
slug: building-accessible-camera-flow-react
featured: false
draft: false
tags:
  - react
  - typescript
  - accessibility
  - web-apis
  - camera
  - ux
description:
  What I learned building a real camera feature in React, from handling
  getUserMedia failures gracefully to stopping media tracks so the camera
  light actually turns off.
---

For our school project **Litter Hero** — an app where users report trash spots around the city so others can clean them up and earn points — I built the in-app camera feature. Users can take a photo directly from the report form, or flip to the front camera, or close the whole thing if they change their mind.

Sounds simple, right? It really wasn't. Here's what I learned.

## TABLE OF CONTENTS

## getUserMedia Fails More Than You Think

The first thing I reached for was `navigator.mediaDevices.getUserMedia()`. It returns a promise that resolves with a `MediaStream` — or rejects with an error. And it _will_ reject. A lot.

Here are the real-world ways it fails, according to [MDN](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia):

- **`NotAllowedError`** — the user denied permission, or the page is loaded over HTTP (not HTTPS). In an insecure context, `navigator.mediaDevices` is literally `undefined`.
- **`NotFoundError`** — no camera found on the device.
- **`NotReadableError`** — the camera exists but is already in use by another app (super common on Windows).
- **`TypeError`** — your constraints object is empty or all `false`.

My first version just... silently failed. The screen went black. Not great when you're asking someone to photograph a pile of trash on the street.

So I added a proper error state. In the component, the `catch` block sets a `cameraError` string:

```tsx
try {
  const stream = await navigator.mediaDevices.getUserMedia({
    video: {
      facingMode: mode,
      width: { ideal: 1920 },
      height: { ideal: 1080 },
    },
    audio: false,
  });
  streamRef.current = stream;
  if (videoRef.current) {
    videoRef.current.srcObject = stream;
  }
} catch {
  setCameraError(
    "Could not access camera. Please check your browser permissions and try again."
  );
}
```

And when `cameraError` is set, instead of a broken black screen, the user sees a clear message and a "Go back" button:

```tsx
{cameraError ? (
  <div className="flex flex-col items-center justify-center h-full text-white p-8 gap-6">
    <svg .../>
    <p className="text-center text-lg">{cameraError}</p>
    <button onClick={onClose} className="px-8 py-3 rounded-full bg-white text-black font-semibold text-sm">
      Go back
    </button>
  </div>
) : (
  /* camera UI */
)}
```

A broken UI is not a neutral experience — it's a trust-breaker. Especially in an app where you're asking people to contribute to something. If the camera just goes black, they'll assume the app is broken, not their browser settings.

## The Camera Isn't Instantly Ready

Another thing I didn't expect: even after `getUserMedia` resolves successfully, the video feed isn't immediately ready to capture. There's a brief moment where the stream is connected but the video element hasn't started rendering frames yet. If the user taps the capture button during that window, they'd get a blank image.

The fix was `onCanPlay` — a native video event that fires when the browser has enough data to start playing. I use it to gate the `isReady` state:

```tsx
const [isReady, setIsReady] = useState(false)

// On the video element:
<video
  ref={videoRef}
  autoPlay
  playsInline
  muted
  onCanPlay={() => setIsReady(true)}
  className="w-full h-full object-cover"
/>
```

While `isReady` is false, a spinner shows over the video, and the capture button is disabled:

```tsx
{
  !isReady && (
    <div className="absolute inset-0 flex items-center justify-center">
      <div className="h-10 w-10 animate-spin rounded-full border-4 border-white/30 border-t-white" />
    </div>
  );
}

<button
  onClick={handleCapture}
  disabled={!isReady}
  aria-label="Take photo"
  className="... transition-transform active:scale-95 disabled:opacity-40"
/>;
```

The `disabled:opacity-40` gives visual feedback that the button isn't interactive yet — no tooltip needed, the dimmed state communicates it.

## Stopping the Stream (This One Really Matters)

This is the detail that separates a demo from a real feature.

When a user closes the camera, the `MediaStream` doesn't stop on its own. The browser keeps the camera active — and on most devices, the little camera indicator light stays on. That's a privacy issue, and on mobile, it's a battery issue too.

The key insight: **you don't stop a `MediaStream` directly. You stop its tracks.**

As [w3tutorials](https://www.w3tutorials.net/blog/stop-close-webcam-stream-which-is-opened-by-navigator-mediadevices-getusermedia/) puts it: _"tracks are the actual 'sources' of media, and they must be explicitly stopped to fully release the device."_ Calling `.stop()` on the stream itself is deprecated and doesn't reliably turn off the camera light.

In the component, I store the stream in a `ref` (not state, because I don't want re-renders when it changes), and clean it up in two places: when the camera flips, and when the component unmounts.

```tsx
const streamRef = useRef<MediaStream | null>(null);

const startCamera = useCallback(async (mode: "environment" | "user") => {
  // Stop any existing tracks before starting new ones
  streamRef.current?.getTracks().forEach(t => t.stop());
  setCameraError(null);
  setIsReady(false);
  // ...start new stream
}, []);

useEffect(() => {
  startCamera(facingMode);
  return () => {
    // Cleanup on unmount
    streamRef.current?.getTracks().forEach(t => t.stop());
  };
}, [facingMode, startCamera]);
```

The `return () => { ... }` inside `useEffect` is React's cleanup function — it runs when the component unmounts or before the effect re-runs. This is what makes sure the camera turns off when the user navigates away, not just when they click "Close".

## Small Accessibility Wins That Add Up

All three camera controls, close, flip, capture — are icon-only buttons. No visible text. That's fine visually, but for screen reader users and keyboard navigators, an icon with no label is just... a button. A mystery button.

The fix is straightforward: `aria-label` on every `<button>`.

```tsx
<button onClick={onClose} aria-label="Close camera" ...>
  <svg .../>
</button>

<button onClick={() => setFacingMode(...)} aria-label="Flip camera" ...>
  <svg .../>
</button>

<button onClick={handleCapture} aria-label="Take photo" ...>
```

As Sara Soueidan explains in her [deep-dive on accessible icon buttons](https://www.sarasoueidan.com/blog/accessible-icon-buttons/): _"when `aria-label` is used on a button, the contents of the attribute will override the contents inside the button as the accessible name"_ — so VoiceOver will announce "Close camera, button" instead of just "button."

Also worth noting: I used real `<button>` elements throughout, not `<div onClick>`. This matters because real buttons are keyboard-focusable by default, support `disabled`, and fire on both click and Enter/Space. You get all of that for free.

## What I'd Add Next

The one thing missing from this flow is a **fallback to file upload** when camera access fails. Right now, the error screen just says "go back." Ideally, it would offer: _"Can't access camera? Upload a photo instead."_ — keeping the user in the flow rather than bouncing them out.

That's the progressive enhancement mindset: don't rely on a single hardware or API path. Not every user will grant camera access, and some devices just don't have one.

That's the next thing I'm adding. 👀

---

## References

- [MediaDevices: getUserMedia() — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)
- [How to Stop or Close Webcam Stream — w3tutorials](https://www.w3tutorials.net/blog/stop-close-webcam-stream-which-is-opened-by-navigator-mediadevices-getusermedia/)
- [Building a Responsive Camera Component with React Hooks — LogRocket](https://blog.logrocket.com/responsive-camera-component-react-hooks/)
- [Accessible Icon Buttons — Sara Soueidan](https://www.sarasoueidan.com/blog/accessible-icon-buttons/)
- [getUserMedia error types by browser — Brian Li (GitHub Gist)](https://gist.github.com/BrianLi101/9bb2f6b8eaf8b24dce93797cd7e788e8)
