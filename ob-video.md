# OB Video Player

OB Video Player is a lightweight, customizable HTML5 video player with a modern interface and advanced features. It's designed to be easily integrated into web projects and offers a range of options for controlling video playback and appearance.

## Table of Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Styling](#styling)
6. [API Reference](#api-reference)
7. [Browser Support](#browser-support)
8. [License](#license)

## Features

- Customizable controls (play/pause, volume, progress bar, fullscreen)
- Lazy loading option for improved performance
- Autoplay and loop options
- Responsive design with customizable aspect ratio
- Keyboard shortcuts for accessibility
- Custom icon support
- Color scheme customization
- Overlay play button for delayed playback
- Loading indicator and error messaging
- Viewport-based play/pause using Intersection Observer
- Pause on tab change option

## Installation

1. Include the OB Video Player JavaScript and CSS files in your HTML:

Head
```html
<link rel="stylesheet" href="https://assets.itsoffbrand.io/utils/ob-video/ob-video.css">
```

Body
```html
<script src="https://assets.itsoffbrand.io/utils/ob-video/ob-video.js"></script>
```

2. Add the necessary HTML structure for your video:

```html
<div ob-video-parent
     ob-video-src1="path/to/video.mp4"
     ob-video-poster="path/to/poster.jpg"
     ob-video-lazy="true"
     ob-video-play-on-enter="true"
     ob-video-pause-on-leave="false"
     ob-video-pause-on-tab-change="true"
     ob-video-loop="false">
</div>
```

## Usage

Initialize the OB Video Player with optional global settings:

```javascript
const options = {
  controls: {
    playButton: true,
    volumeControl: true,
    progressBar: true,
    fullscreenToggle: true,
  },
  icons: {
    // Custom icon URLs (optional)
  },
  colors: {
    // Custom color settings (optional)
  },
};

const obVideoPlayer = new OBVideoPlayer(options);
```

## Configuration

You can configure each video player instance using HTML attributes:

- `ob-video-src1`, `ob-video-src2`, etc.: Video source URLs (multiple sources for different formats)
- `ob-video-poster`: Poster image URL
- `ob-video-lazy`: Enable lazy loading ("true" or "false")
- `ob-video-play-on-enter`: Auto-play when entering viewport ("true" or "false")
- `ob-video-pause-on-leave`: Pause when leaving viewport ("true" or "false")
- `ob-video-pause-on-tab-change`: Pause when changing browser tab ("true" or "false")
- `ob-video-loop`: Enable video looping ("true" or "false")

Custom icons can be set using these attributes:
- `ob-video-icon-play`
- `ob-video-icon-pause`
- `ob-video-icon-volume-on`
- `ob-video-icon-volume-off`
- `ob-video-icon-fullscreen`
- `ob-video-icon-fullscreen-exit`
- `ob-video-icon-overlay-play`

## Styling

OB Video Player uses CSS variables for easy customization. You can override these variables in your own CSS:

```css
:root {
  /* Primary Colors */
  --ob-primary-color: rgba(
    255,
    255,
    255,
    0.916
  ); /* Icon color and accent color */
  --ob-icon-color: #ffffff;
  --ob-background-overlay: rgba(0, 0, 0, 0.5); /* Overlay background */

  /* Button Colors */
  --ob-control-button-bg: transparent;
  --ob-control-button-color: var(--ob-primary-color);
  --ob-control-button-hover-bg: var(--ob-primary-color);
  --ob-control-button-hover-color: #ffffff;
  --ob-control-button-hover-transition: background 0.2s;
  --ob-control-button-border-radius: 4px;

  /* Overlay Play Button */
  --ob-overlay-play-button-bg: var(--ob-background-overlay);
  --ob-overlay-play-button-color: var(--ob-primary-color);

  /* Progress Bar Colors */
  --ob-progress-bar-bg: var(--ob-primary-color); /* Updated to primary color */
  --ob-progress-bar-handle-color: var(--ob-primary-color);

  /* Volume Control Colors */
  --ob-volume-control-bg: var(
    --ob-primary-color
  ); /* Updated to primary color */

  /* Indicator Backgrounds */
  --ob-loading-indicator-bg: rgba(0, 0, 0, 0.7);
  --ob-error-message-bg: rgba(255, 0, 0, 0.7);

  /* Sizes */
  --ob-icon-size: 1rem;
  --ob-button-size: 1.6rem;
  --ob-button-border-radius: 4px;
  --ob-overlay-play-button-size: 4rem;
  --ob-volume-range-width: 100px;

  /* Opacity and Transition */
  --ob-control-opacity: 0;
  --ob-control-hover-opacity: 1;
  --ob-transition-duration: 0.3s;

  /* Aspect Ratio */
  --ob-aspect-ratio: 56.25%; /* Default to 16:9 aspect ratio */

  /* Margins and Padding */
  --ob-control-margin: 0 5px;
  --ob-progress-container-margin: 0 10px;
  --ob-controls-padding: 5px;
  --ob-volume-icon-margin-right: 5px;

  /* Z-Index */
  --ob-overlay-z-index: 10; /* Higher z-index */
}
```

Refer to the CSS file for a complete list of customizable variables.

## API Reference

### OBVideoPlayer

#### Constructor

```javascript
new OBVideoPlayer(options)
```

- `options`: An object containing global settings for all video players.

#### Methods

- `destroy()`: Removes all event listeners and cleans up resources.

### HTML Attributes

- `ob-video-parent`: Designates the container for a video player instance.
- `ob-video-src1`, `ob-video-src2`, etc.: Specify video sources.
- `ob-video-poster`: Set the poster image.
- `ob-video-lazy`: Enable lazy loading.
- `ob-video-play-on-enter`: Enable autoplay when entering viewport.
- `ob-video-pause-on-leave`: Enable pause when leaving viewport.
- `ob-video-pause-on-tab-change`: Enable pause on tab change.
- `ob-video-loop`: Enable video looping.

## License

MIT
