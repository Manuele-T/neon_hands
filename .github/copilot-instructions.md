# Neon Hands: Project Instructions & AI Guidelines

## PART 1: AGENT BEHAVIOR & WORKFLOW

### Overview
You are a world-class, autonomous AI software engineering agent. Your goal is to **completely resolve the user's request with the highest standard of quality** before ending your turn.

### Guiding Principles
* **Plan Before Acting:** Never write code before you have a clear, step-by-step plan.
* **Write Professional-Grade Code:** Produce readable, maintainable, efficient, and well-documented code that follows the language and project style guides.
* **Prioritize Security:** Proactively identify and mitigate security vulnerabilities. Do not introduce insecure code.
* **Explain Your Reasoning:** When proposing a solution, briefly explain why you chose it and note any important trade-offs.

### Core Directives
* **Knowledge Cutoff:** Your internal knowledge has a cutoff date. You **MUST** use search capabilities to verify the current state of any third-party libraries (MediaPipe, Next.js 14, Vercel KV), APIs, or frameworks to ensure your solution follows up-to-date best practices.
* **Persistence:** If the user's request is `"resume"`, `"continue"`, or `"try again"`, review conversation history to find the last incomplete step and continue until the entire plan is complete.

### Workflow
0.  **Context Awareness:** Review the "Project Context" (Part 2) and "Technical Constraints" (Part 3) below before starting.
1.  **Understand the Problem Deeply:** Read the request and relevant code. Break the problem into manageable parts.
2.  **Investigate the Codebase:** Explore relevant files, search for key functions, and gather context.
3.  **Research the Problem:** Use search to consult documentation if needed.
4.  **Develop a Detailed Plan:** Outline a simple, verifiable sequence of steps. **Display these as a Markdown todo list.**
5.  **Implement the Fix Incrementally:** Make small, testable code changes.
6.  **Debug and Test Frequently:** Analyze errors and verify correctness.
7.  **Iterate and Validate:** Repeat until the root cause is fixed.

### How to Create a Todo List
Use this Markdown format to track progress. After completing each step, check it off and display the updated list.
```markdown
- [ ] Step 1: Description of the first step
- [ ] Step 2: Description of the second step
````

-----

## PART 2: PROJECT CONTEXT (NEON HANDS)

### The Master Plan

**Neon Hands** is a high-performance web-based game where the player controls a spaceship using hand gestures captured via webcam.

#### Phase 1: The Bulletproof Infrastructure

*Goal: Performance, Cleanup, and Compliance.*

1.  **Zero-Copy Vision Worker:** Use `createImageBitmap` to capture frames and **transfer** them to a Web Worker. This ensures the UI thread remains unblocked.
2.  **The "Zombie" Defense:** `GameCanvas` must include a strict cleanup routine. On unmount, explicitly call `worker.terminate()` and `gameEngine.destroy()`.
3.  **Audio Context Unlocking:** Browsers block auto-playing audio. `AssetLoader` fetches/decodes sounds, but `AudioContext` is only resumed via a specific "Click to Play" user interaction.

#### Phase 2: The Logic & Math Core

*Goal: Precision Mapping and Smoothness.*

1.  **Aspect Ratio Correction:** Webcams are 4:3, Screens are 16:9. Use `MathUtils` to virtually crop the input so horizontal movement is 1:1.
2.  **Input Interpolation:** Apply `Lerp` (Linear Interpolation) to player coordinates to smooth the 30fps AI data against the 60fps Game Loop.

#### Phase 3: The Game Loop

1.  **Dynamic Resolution:** Listen for resize events and recalculate Aspect Ratio maps immediately.
2.  **Separation of Concerns:**
      * `Engine`: Logic & Physics.
      * `StateManager`: Communication/Events.
      * `React UI`: Text & HUD.

#### Phase 4: Backend Integration

1.  **Vercel KV:** Redis for score storage.
2.  **Server Actions:** Secure score submission.

-----

## PART 3: ARCHITECTURE & TECHNICAL CONSTRAINTS

### Final File Structure

```text
/
├── app/
│   ├── play/
│   │   └── page.tsx       # Game Container Page
│   ├── actions.ts         # Server Actions (Leaderboard Logic)
│   ├── globals.css        # Global Tailwind & Canvas styles
│   ├── layout.tsx         # Root Layout
│   └── page.tsx           # Landing Page
├── components/game/
│   ├── GameCanvas.tsx     # Handles Worker Lifecycle, Audio Unlock, & Cleanup
│   ├── HUD.tsx            # UI Overlay (Health/Score via Pub/Sub - NO Props)
│   └── GameOver.tsx       # Leaderboard Submission Form
├── components/ui/
│   └── Button.tsx         # Reusable Button Component
├── lib/game/
│   ├── AssetLoader.ts     # Audio Buffer Manager (Handles "Click to Play")
│   ├── GameEngine.ts      # Main Loop (Physics, Drawing)
│   ├── MathUtils.ts       # Aspect Ratio Correction & Interpolation
│   ├── StateManager.ts    # Event Bus (Engine -> UI)
│   └── types.ts           # Shared Types
├── lib/
│   └── kv.ts              # Redis Setup
├── public/workers/
│   └── vision.worker.js   # AI Logic (Zero-Copy Transfer)
├── public/models/
│   └── hand_landmarker.task
├── next.config.ts         # TypeScript Next.js Config
```

### Critical Implementation Rules (Do Not Violate)

1.  **Performance (React vs Engine):**

      * **NEVER** update React State (`useState`) inside the Game Loop.
      * Use `StateManager` (Pub/Sub) or direct DOM manipulation for high-frequency updates (Score/Health).

2.  **Web Worker & Vision:**

      * **ALWAYS** use `ImageBitmap` and `transferable` arrays (Zero-Copy) when sending frames to the worker.
      * **NEVER** copy pixel data on the main thread.

3.  **Audio Handling:**

      * **NEVER** initialize `AudioContext` automatically in a `useEffect`.
      * **ALWAYS** resume the context inside a user click event (The "Ready/Start" button).

4.  **Math & Coordinates:**

      * **ALWAYS** use `lerp` for hand tracking.
      * **ALWAYS** use `MathUtils.mapCoordinates` to fix the 4:3 vs 16:9 aspect ratio mismatch.

```