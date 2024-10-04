# OB Cookie Consent Implementation

This code is not fully GDPR complient, but mostly.

## Features

- Display a cookie consent banner to first-time visitors
- Allow users to accept or reject cookies
- Enable or disable Google Analytics tracking based on user choice
- Persist user choice using localStorage
- Provide a way for users to modify their cookie preferences
- Animate the cookie banner using GSAP (GreenSock Animation Platform)

## Code Explanation

The main function `initCookies()` sets up the cookie consent mechanism. Here's a breakdown of its components:

1. **DOM Elements**: The code selects various elements from the DOM, including the cookie banner, accept/reject buttons, and modify buttons.

2. **Constants**: 
   - `COOKIE_FLAG_KEY`: The key used to store the user's cookie preference in localStorage.
   - `GA_MEASUREMENT_ID`: The Google Analytics measurement ID.

3. **Helper Functions**:
   - `getCookieFlag()`: Retrieves the stored cookie preference.
   - `setCookieFlag()`: Stores the user's cookie preference.
   - `animateIn()` and `animateOut()`: Animate the cookie banner using GSAP.
   - `handleCookieResponse()`: Handles the user's response (accept or reject), updates preferences, and enables/disables tracking.
   - `disableGATracking()`: Disables Google Analytics tracking and clears existing cookies.
   - `enableGATracking()`: Enables Google Analytics tracking.

4. **Initialization**:
   - `initializeTracking()`: Checks for existing preferences and initializes the banner and tracking accordingly.

5. **Event Listeners**: Sets up event listeners for the accept, reject, and modify buttons.

## Full Code

```javascript
function initCookies() {
  const cookieBanner = document.querySelector('[data-cookie="parent"]');
  const acceptButton = cookieBanner?.querySelector('[data-cookie="accept"]');
  const rejectButton = cookieBanner?.querySelector('[data-cookie="reject"]');
  const modifyButtons = document.querySelectorAll('[data-cookie="modify"]');
  const COOKIE_FLAG_KEY = "cookiesAccepted";
  const GA_MEASUREMENT_ID = "ADD_ID_HERE";

  function getCookieFlag() {
    return localStorage.getItem(COOKIE_FLAG_KEY);
  }

  function setCookieFlag(value) {
    localStorage.setItem(COOKIE_FLAG_KEY, value.toString());
  }

  function animateIn() {
    gsap.to(cookieBanner, {
      y: "0rem",
      autoAlpha: 1,
      duration: 0.6,
      ease: "power2.inOut",
    });
  }

  function animateOut() {
    gsap.to(cookieBanner, {
      y: "5rem",
      autoAlpha: 0,
      duration: 0.6,
      ease: "power2.inOut",
    });
  }

  function handleCookieResponse(accepted) {
    setCookieFlag(accepted);
    animateOut();
    if (accepted) {
      enableGATracking();
    } else {
      disableGATracking();
    }
  }

  function disableGATracking() {
    window[`ga-disable-${GA_MEASUREMENT_ID}`] = true;
    // Clear any existing cookies
    document.cookie = "_ga=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
    document.cookie = "_gat=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
    document.cookie = "_gid=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
  }

  function enableGATracking() {
    window[`ga-disable-${GA_MEASUREMENT_ID}`] = false;
  }

  // Implement consent before tracking
  function initializeTracking() {
    const cookieFlag = getCookieFlag();
    if (cookieFlag === null) {
      disableGATracking(); // Disable tracking by default
      animateIn(); // Show the banner for first-time visitors
    } else {
      gsap.set(cookieBanner, { autoAlpha: 0, y: "5rem" });
      if (cookieFlag === "true") {
        enableGATracking();
      } else {
        disableGATracking();
      }
    }
  }

  // Add event listeners to buttons
  acceptButton?.addEventListener("click", () => handleCookieResponse(true));
  rejectButton?.addEventListener("click", () => handleCookieResponse(false));

  // Add event listeners to modify buttons
  modifyButtons.forEach(button => {
    button.addEventListener("click", () => {
      animateIn(); // Show the banner again when modify is clicked
    });
  });

  // Initialize tracking with consent
  initializeTracking();
}

initCookies();
```

## Usage

To use this cookie consent mechanism:

1. Include the GSAP library in your project.
2. Add the necessary HTML elements with the appropriate `data-cookie` attributes.
3. Include this JavaScript code in your project.
4. Call `initCookies()` when the DOM is ready.

Make sure to replace the `GA_MEASUREMENT_ID` constant with your actual Google Analytics measurement ID.

## Customization

You can customize the appearance in your HTML. The animation can be adjusted by modifying the GSAP animation parameters in the `animateIn()` and `animateOut()` functions.
