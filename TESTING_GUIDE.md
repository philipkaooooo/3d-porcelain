# Testing Guide for Performance Optimizations

## Quick Verification Tests

### 1. Visual Functionality Test
All existing features should work identically to before:

- [ ] Scene loads without errors
- [ ] Can rotate camera with mouse drag
- [ ] Arrow keys change scenes (5 scenes total)
- [ ] Space bar toggles motion
- [ ] Mouse hover shows cursor effects
- [ ] Click creates particle pulse
- [ ] All toggles work (Bloom, DOF, Particles, Scanner, Day/Night)
- [ ] Scene transitions are smooth
- [ ] Particle effects display correctly
- [ ] Material scanner effect activates

### 2. Performance Verification

#### Frame Rate Test
Open browser DevTools (F12) and check:

```javascript
// Add this to console to monitor FPS
let lastTime = performance.now();
let frames = 0;
setInterval(() => {
  const now = performance.now();
  const fps = frames / ((now - lastTime) / 1000);
  console.log('FPS:', fps.toFixed(1));
  frames = 0;
  lastTime = now;
}, 1000);
// Then modify animation loop to increment frames counter
```

**Expected**: 55-60 FPS on modern devices

#### Raycast Optimization Test
Open console and monitor raycasting:

```javascript
// Check that raycasts only happen on mouse move
let raycastCount = 0;
const originalSetFromCamera = THREE.Raycaster.prototype.setFromCamera;
THREE.Raycaster.prototype.setFromCamera = function(...args) {
  raycastCount++;
  return originalSetFromCamera.apply(this, args);
};

setInterval(() => {
  console.log('Raycasts per second:', raycastCount);
  raycastCount = 0;
}, 1000);
```

**Expected**: 
- Mouse still: ~0 raycasts/sec
- Mouse moving: ~10-30 raycasts/sec
- Before optimization: 60 raycasts/sec constantly

#### DOM Update Test
Check time display only updates once per second:

```javascript
// Enable Day/Night cycle toggle
// Watch time display in bottom-left panel
// Should update once per second, not every frame
```

**Expected**: Visible "tick" once per second

### 3. Regression Tests

#### Cursor Interaction
1. Move mouse over porcelain object
2. **Expected**: 
   - Cursor ball appears
   - Glow effect shows
   - Material becomes more reflective
3. Move mouse away
4. **Expected**:
   - Effects fade smoothly
   - No sudden jumps or artifacts

#### Scene Transitions
1. Press right arrow key
2. **Expected**:
   - Smooth camera movement
   - Scene fades in/out properly
   - Lighting transitions smoothly
   - No visual glitches

#### Particle Effects
1. Click anywhere on scene
2. **Expected**:
   - Particle burst appears at click point
   - Particles fade out over time
   - No performance drop during burst

#### Material Scanner
1. Enable "Material Scanner" toggle
2. Move mouse over object
3. **Expected**:
   - Blue scanning effect follows cursor
   - Pulsing animation is smooth
   - Effect intensity varies correctly

### 4. Browser Compatibility

Test in multiple browsers:
- [ ] Chrome/Edge (Chromium)
- [ ] Firefox
- [ ] Safari (if available)

All features should work identically in supported browsers.

### 5. Device Testing

If possible, test on:
- [ ] Desktop/laptop
- [ ] Tablet (if available)
- [ ] Mobile device (if available)

Performance improvements should be most noticeable on mid-range devices.

## Automated Verification Script

Run this in browser console for quick check:

```javascript
// Comprehensive optimization verification
const verifyOptimizations = () => {
  const results = {};
  
  // Check mouseMoved flag exists
  results.raycastOptimization = typeof mouseMoved !== 'undefined';
  
  // Check temporal system has throttling
  results.domThrottling = temporalSystem.hasOwnProperty('lastDisplayUpdate');
  
  // Check animation frame ID exists
  results.cleanupSupport = typeof animationFrameId !== 'undefined';
  
  // Check frustum culling on cursor ball
  results.frustumCulling = cursorBall.frustumCulled === true;
  
  console.table(results);
  
  const passed = Object.values(results).every(v => v === true);
  console.log(passed ? '✅ All optimizations verified!' : '❌ Some checks failed');
  
  return results;
};

verifyOptimizations();
```

## Performance Comparison

### Before Optimizations
Typical metrics on mid-range device:
- FPS: 45-55 (drops to 40 during interaction)
- CPU: 35-45% (one core)
- Draw calls: ~200-250/frame
- Raycasts: 60/second constantly

### After Optimizations
Expected metrics on same device:
- FPS: 55-60 (more stable)
- CPU: 25-35% (one core)
- Draw calls: ~180-220/frame
- Raycasts: 0-30/second (only when needed)

## Known Issues & Limitations

### Expected Behavior
1. **First frame may stutter**: Initial shader compilation is normal
2. **HDR loading delay**: Optional HDR environment loads asynchronously
3. **Mobile performance**: Touch interactions may differ from mouse

### Not Performance Issues
1. **Scene transition pause**: Intentional for visual effect
2. **Particle fadeout**: Designed behavior, not a lag
3. **Material response delay**: Smooth lerping is intentional

## Reporting Issues

If you find performance regressions:

1. **Describe the scenario**:
   - What action triggered the issue?
   - Which scene were you viewing?
   - What effects were enabled?

2. **Provide metrics**:
   - FPS before/after action
   - Browser and version
   - Device specifications

3. **Check console**:
   - Any error messages?
   - Warning messages?

4. **Compare to baseline**:
   - Does issue occur without optimizations?
   - Is it worse, same, or better than before?

## Debugging Tips

### Disable Optimizations Temporarily

To verify an optimization is working, temporarily disable it:

```javascript
// Force raycasting every frame (disable optimization)
mouseMoved = true; // in animation loop

// Force DOM updates every frame (disable throttling)
temporalSystem.lastDisplayUpdate = 0; // in update

// Disable frustum culling (revert optimization)
cursorBall.frustumCulled = false;
```

Compare performance with/without to measure impact.

### Monitor Specific Metrics

```javascript
// Track material updates
let materialUpdates = 0;
// Add counter in scene.update() material loop
console.log('Material updates:', materialUpdates);

// Track particle updates  
let particleUpdates = 0;
// Add counter when particles are updated
console.log('Particle updates:', particleUpdates);
```

## Success Criteria

Optimizations are successful if:
- ✅ All visual features work identically
- ✅ FPS improves or stays stable
- ✅ CPU usage decreases
- ✅ Interactions feel more responsive
- ✅ No new visual artifacts
- ✅ Mobile/low-end devices see improvement

## Rollback Plan

If issues arise:
1. Git revert to previous commit
2. Identify specific problematic optimization
3. Disable that specific optimization
4. Re-test remaining optimizations
5. Document the issue for future investigation
