## requestAnimationFrame Basics
```javascript
// Basic Animation Loop
function animate(timestamp) {
    // Animation logic here
    requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Canceling Animation
const animationId = requestAnimationFrame(animate);
cancelAnimationFrame(animationId);
```
`requestAnimationFrame` is a browser API that schedules a function to run before the next repaint. It's the modern replacement for `setTimeout/setInterval` for animations. The browser optimizes these calls for performance and battery life, and automatically pauses them when the tab is inactive. Returns an ID that can be used to cancel the animation.

## Timing and Progress Calculations
```javascript
// Basic Timing
let startTime = null;

function animate(timestamp) {
    if (!startTime) startTime = timestamp;
    const elapsed = timestamp - startTime;
    requestAnimationFrame(animate);
}

// Progress Ratio (0 to 1)
const DURATION = 1000; // milliseconds
const progress = Math.min((timestamp - startTime) / DURATION, 1);

// Looping Animation
const progress = ((timestamp - startTime) % DURATION) / DURATION;
```
These are fundamental calculations for animation timing. The `timestamp` is provided by the browser and represents milliseconds since page load. The progress ratio converts elapsed time into a 0-1 value useful for interpolation. The modulo operator (%) creates repeating animations by wrapping the time back to 0.

## Easing Functions
```javascript
// Common Easing Functions
const easings = {
    linear: t => t,
    easeInQuad: t => t * t,
    easeOutQuad: t => t * (2 - t),
    easeInOutQuad: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,
    easeInCubic: t => t * t * t,
    easeOutCubic: t => (--t) * t * t + 1,
    easeInOutCubic: t => t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1
};
```
Easing functions modify the rate of change of an animation. They take a linear input (0-1) and return a modified value that creates various acceleration effects. These mathematical functions help create more natural-feeling animations. For example, 'easeInQuad' starts slow and accelerates.

## React Animation Hook
```javascript
function useAnimation(duration, callback) {
    useEffect(() => {
        let startTime = null;
        let animationFrameId;

        function animate(timestamp) {
            if (!startTime) startTime = timestamp;
            const progress = Math.min((timestamp - startTime) / duration, 1);
            
            callback(progress);

            if (progress < 1) {
                animationFrameId = requestAnimationFrame(animate);
            }
        }

        animationFrameId = requestAnimationFrame(animate);
        return () => cancelAnimationFrame(animationFrameId);
    }, [duration, callback]);
}

// Usage
useAnimation(1000, (progress) => {
    element.style.opacity = progress;
});
```
A custom React hook that handles animation setup and cleanup. It uses `useEffect` to manage the animation lifecycle and automatically cleans up when the component unmounts. This pattern prevents memory leaks and follows React best practices for side effects.

## Common Animation Patterns

### Fade In/Out
```javascript
function fadeIn(element, duration = 1000) {
    let startTime = null;
    
    function animate(timestamp) {
        if (!startTime) startTime = timestamp;
        const progress = Math.min((timestamp - startTime) / duration, 1);
        
        element.style.opacity = progress;
        
        if (progress < 1) {
            requestAnimationFrame(animate);
        }
    }
    
    requestAnimationFrame(animate);
}
```
A reusable function for fading elements in. Uses opacity which is one of the most performant properties to animate. The browser can optimize opacity changes better than many other visual properties.

### Smooth Scrolling
```javascript
function smoothScroll(target, duration = 1000) {
    const start = window.pageYOffset;
    const distance = target - start;
    let startTime = null;

    function animate(timestamp) {
        if (!startTime) startTime = timestamp;
        const progress = Math.min((timestamp - startTime) / duration, 1);
        
        window.scrollTo(0, start + distance * progress);
        
        if (progress < 1) {
            requestAnimationFrame(animate);
        }
    }
    
    requestAnimationFrame(animate);
}
```
A smooth scrolling implementation that's more customizable than the built-in `scrollTo` behavior. `pageYOffset` is a browser property that gives the current scroll position. Note: Modern browsers now have `scrollTo({ behavior: 'smooth' })` built-in.

## Performance Tips
```javascript
// Use CSS Transform instead of top/left
element.style.transform = `translateX(${x}px)`;  // Good
element.style.left = `${x}px`;                  // Bad

// Batch DOM Updates
requestAnimationFrame(() => {
    element1.style.transform = '...';
    element2.style.opacity = '...';
    element3.style.scale = '...';
});

// Use will-change for optimization
element.style.willChange = 'transform';
```
CSS transforms use GPU acceleration and don't trigger layout recalculations. Batching DOM updates within requestAnimationFrame prevents multiple repaints. `will-change` hints to browsers about properties likely to change, but should be used sparingly as it can consume memory.

## Debug Helpers
```javascript
// FPS Counter
let frameCount = 0;
let lastTime = performance.now();

function countFPS(timestamp) {
    frameCount++;
    
    if (timestamp - lastTime >= 1000) {
        console.log(`FPS: ${frameCount}`);
        frameCount = 0;
        lastTime = timestamp;
    }
    
    requestAnimationFrame(countFPS);
}

requestAnimationFrame(countFPS);
```
A simple FPS (Frames Per Second) counter using `performance.now()` for high-precision timing. Useful for debugging animation performance. The browser typically targets 60 FPS, so values consistently below that indicate performance issues.

## Key Points & Gotchas
- `requestAnimationFrame` is supported in all modern browsers (IE10+)
- Animation timing is handled by the browser's internal scheduler
- Heavy calculations should be cached outside the animation loop
- CSS animations are often more performant for simple transitions
- Mobile devices may require simplified animations for performance
- Always test animations on lower-end devices
- Memory leaks can occur if animations aren't properly cleaned up
- The browser may throttle animations in background tabs