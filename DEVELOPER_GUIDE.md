# Developer Quick Reference - Mindly App

## Using the Unified Design System

### Import Game UI Constants
```typescript
import { GameUIClasses, GameUIConstants, normalizeScore, calculateAccuracy } from "@/hooks/useGameUI";
```

### Common Class Patterns

#### Game Wrapper Structure
```tsx
<div className={`${GameUIClasses.wrapper} ${GameUIClasses.wrapperPadding}`}>
  {/* Your game content */}
</div>
```

#### Stat Display
```tsx
<div className={GameUIClasses.scoreDisplay}>
  <p className={GameUIClasses.scoreLabel}>LEVEL</p>
  <p className={GameUIClasses.scoreValue}>{level}</p>
</div>
```

#### Interactive Button
```tsx
<button className={GameUIClasses.interactiveButton}>
  {/* Button content */}
</button>
```

#### Progress Bar
```tsx
<div className={GameUIClasses.progressBar}>
  <motion.div 
    className={GameUIClasses.progressFill} 
    animate={{ width: `${progress}%` }} 
  />
</div>
```

## Standard Game Pattern

### Component Structure
```typescript
import { useState, useEffect, useCallback } from "react";
import { motion } from "framer-motion";
import { GameResult } from "@/store/useGameStore";
import GameWrapper from "./GameWrapper";
import { useSound } from "@/hooks/useSound";
import { GameUIClasses, normalizeScore, calculateAccuracy } from "@/hooks/useGameUI";

export default function MyGame({ onComplete }: { onComplete: (result: GameResult) => void }) {
  const [phase, setPhase] = useState<"playing" | "done">("playing");
  const [score, setScore] = useState(0);
  const [timeLeft, setTimeLeft] = useState(30);
  
  // Timer logic
  useEffect(() => {
    if (phase !== "playing") return;
    const timer = setInterval(() => {
      setTimeLeft(t => t <= 1 ? 0 : t - 1);
    }, 1000);
    return () => clearInterval(timer);
  }, [phase]);

  // Game end
  useEffect(() => {
    if (phase === "playing" && timeLeft <= 0) {
      setPhase("done");
      onComplete({
        gameType: "my-game",
        score: normalizeScore(score, 100),
        accuracy: calculateAccuracy(correctCount, totalCount),
        details: { /* your metrics */ },
        timestamp: Date.now()
      });
    }
  }, [timeLeft, phase, score, onComplete]);

  return (
    <GameWrapper title="My Game" description="Game description" onReady={() => {}}>
      <div className={`${GameUIClasses.wrapper} ${GameUIClasses.wrapperPadding}`}>
        {/* Game content */}
      </div>
    </GameWrapper>
  );
}
```

## Scoring Standards

### Normalize Scores (Always use this)
```typescript
// Instead of Math.min(100, Math.round((score / max) * 100))
// Use:
score: normalizeScore(rawScore, maxPossibleScore)

// Examples:
normalizeScore(350, 350) // → 100
normalizeScore(175, 350) // → 50
normalizeScore(52.5, 350) // → 15
```

### Calculate Accuracy
```typescript
// Instead of ternary checks
// Use:
accuracy: calculateAccuracy(correctAnswers, totalAttempts)

// Examples:
calculateAccuracy(10, 10) // → 100%
calculateAccuracy(7, 10) // → 70%
```

## Color Classes

Use these for dynamic colors:
```typescript
// Primary (blue)
className="text-primary bg-primary/5 border-primary"

// Success (green)
className="text-green-500 bg-green-500/10"

// Warning (yellow)
className="text-yellow-500 bg-yellow-500/10"

// Error (red)
className="text-red-500 bg-red-500/10"
```

## Animation Standards

### Standard Durations
```typescript
// 150ms - Fast feedback (clicks, hovers)
className="transition-all duration-150 ease-out"

// 300ms - Normal transitions (fade, scale)
className="transition-all duration-300 ease-out"

// 500ms - Slow animations (page load, intro)
className="transition-all duration-500 ease-out"

// 1000ms - Very slow (cinematic effects)
className="transition-all duration-1000 ease-out"
```

## Game Result Format

All games must return this format:
```typescript
{
  gameType: "focus" | "reaction" | "memory" | "distraction" | "impulse",
  score: 0-100 (normalized),
  accuracy: 0-100,
  reactionTime?: number (milliseconds),
  details: { /* any additional metrics */ },
  timestamp: Date.now()
}
```

## Best Practices

1. **Always normalize scores** to 0-100 range
2. **Always calculate accuracy** consistently  
3. **Always use GameUIClasses** for styling
4. **Always clean up intervals** in useEffect return
5. **Always set phase to "done"** before calling onComplete
6. **Always use GameWrapper** for consistent header/footer
7. **Always use motion from framer-motion** for animations
8. **Always import from @/hooks/useGameUI** for consistency

## Common Mistakes to Avoid

❌ **Bad**: `Math.min(100, Math.round((score / 300) * 100))`
✅ **Good**: `normalizeScore(score, 300)`

❌ **Bad**: `Math.round((hits / (hits + misses)) * 100) || 0`
✅ **Good**: `calculateAccuracy(hits, hits + misses)`

❌ **Bad**: Custom class combinations for every element
✅ **Good**: Use GameUIClasses constants

❌ **Bad**: Different timer implementations per game
✅ **Good**: Use <GameTimer /> component

❌ **Bad**: Missing useEffect cleanups
✅ **Good**: Always return cleanup function from setInterval/setTimeout

## File Locations

- **UI Constants**: `src/hooks/useGameUI.ts`
- **Game Components**: `src/components/games/`
- **Game Store**: `src/store/useGameStore.ts`
- **Sound Hook**: `src/hooks/useSound.ts`
- **Game Wrapper**: `src/components/games/GameWrapper.tsx`
- **Timer Component**: `src/components/GameTimer.tsx`

## Quick Commands

```bash
# Check linting
npm run lint

# Run in development
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run tests
npm run test

# Watch tests
npm run test:watch
```

---

**Remember**: The goal is consistency across all games. Always refer to this guide and existing games as templates!
