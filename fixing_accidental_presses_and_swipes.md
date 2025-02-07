# Summary of React Native Gesture & Carousel Discussion

We’ve been troubleshooting an issue with large buttons and a horizontally swipable carousel (via **`react-native-reanimated-carousel`**) inside a vertically scrollable page in React Native. Below is a summary of what we discussed and the solutions we identified.

---

## Problem 1: Accidental Button Presses During Vertical Scrolling

### Issue
When a user tries to scroll vertically, a press on large, touchable buttons is sometimes registered instead of a scroll gesture.

### Recommended Solutions
1. **`pressRetentionOffset`**  
   - Increase this prop on your pressable components (e.g., `TouchableOpacity`) so that any finger movement (scroll attempt) beyond a certain threshold cancels the press.  
   - Example:  
     ```jsx
     <TouchableOpacity
       onPress={...}
       pressRetentionOffset={{ top: 20, bottom: 20, left: 20, right: 20 }}
       delayPressIn={50} // optional small delay
     >
       { /* Button content */ }
     </TouchableOpacity>
     ```

2. **`delayPressIn`**  
   - Introducing a short delay (e.g., 50 ms) before a press is recognized allows time to detect the user’s scroll vs. a true tap.

3. **Advanced: `react-native-gesture-handler`**  
   - In more complex scenarios (e.g., nested scroll views or advanced swipe features), leveraging `PanGestureHandler` or `TapGestureHandler` with `waitFor` and `simultaneousHandlers` can give finer control.  
   - This is often overkill if `pressRetentionOffset` and `delayPressIn` solve the issue.

---

## Problem 2: Carousel Takes Over Vertical Scrolling

### Issue
Inside a vertically scrolling page, a small vertical movement accidentally triggers horizontal swipes on the carousel, making the carousel scroll sideways instead of letting the user scroll up and down.

### Recommended Solution

**Use `panGestureHandlerProps` in `react-native-reanimated-carousel`**  
- The key is to set thresholds (`activeOffsetX` and `failOffsetY`) that define how much horizontal or vertical movement is needed to activate or fail the carousel swipe.

#### Example:

```jsx
<Carousel
  width={0}
  height={120}
  data={infoCardData}
  renderItem={({ item }) => <InfoCard {...item} />}
  // The magic props
  panGestureHandlerProps={{
    // Require at least 10px horizontal movement to activate the carousel
    activeOffsetX: [-10, 10],
    // If the user drags more than 10px vertically, let the parent scroll have priority
    failOffsetY: [-10, 10],
  }}
  pagingEnabled
  snapEnabled
/>
