# Building a Real-Time AR Drawing App with Snap Spectacles, TypeScript, and Supabase

Augmented reality (AR) wearables like Snap Spectacles are ushering in a new era of spatial computing—one you can program with TypeScript and powerful web services. In this post, we’ll walk through building an interactive AR drawing app for Spectacles that leverages hand gestures, speech commands, and real-time data sharing with Supabase (via SnapCloud).

## Overview

- **What You'll Build**: An AR drawing app for Snap Spectacles. Draw ASCII characters in midair with gestures, clear the canvas with your voice, and broadcast everything in real-time to a web app. Persist drawings to a database, and trigger image generation with serverless edge functions.
- **Tech Stack**: Snap Lens Studio, TypeScript, SnapCloud (Supabase), basic web frontend

---

## Prerequisites

- Snap Spectacles (Next Gen) and Snap Lens Studio installed
- TypeScript experience
- Supabase/SnapCloud account access
- Web development tools (Node.js, npm, basic frontend familiarity)

---

## Step 1: Setting Up the AR Environment in Lens Studio

1. **Download and Install Lens Studio**
   - [Lens Studio Download](https://ar.snap.com/lens-studio)
2. **Create a New Project for Spectacles**
   - Select the Spectacles target during project creation.
3. **Add 3D Assets to the Scene**
   - Insert a 3D Text label prefab (such as a smiley face or any text).
   - Adjust its position and styling as desired in the 3D workspace.

4. **Prepare Assets for Reuse**
   - Change font/style as needed.
   - Drag the customized 3D text back into the Assets panel to create a prefab.

---

## Step 2: Adding Interactivity with TypeScript Scripting

1. **Create a TypeScript Module**
   - In Lens Studio, create a new TypeScript file (e.g., `module1.ts`).

2. **Edit the Script**
   - Extend the base script component.
   - Implement `onAwake()` to handle initialization.

   ```typescript
   export class DrawingComponent extends BaseScriptComponent {
     onAwake() {
       print('Hello from specs');
     }
   }
   ```

3. **Add Hand Tracking Logic**
   - Reference Snap’s documentation for code samples.
   - Track left/right hands and handle pinch gestures.

   ```typescript
   // Pseudocode for detecting pinch gesture
   this.handTrackingModule.onPinch((event) => {
     print(`Pinch detected at ${event.position}`);
   });
   ```

   > **Note:** Snap provides built-in modules for hand tracking. Make sure these are included in your project dependencies.

4. **Create and Place Prefabs on Gesture**
   - When pinching with the right hand, instantiate a 3D label prefab at the index finger tip.

   ```typescript
   const label = this.prefab.instantiate();
   label.transform.setWorldPosition(event.position);
   // Add to array for tracking
   this.labels.push(label);
   ```

5. **Implement Scene Reset**
   - Store all generated labels in an array.
   - Add a `newScene()` function to destroy and remove all labels.

   ```typescript
   newScene() {
     this.labels.forEach(label => label.destroy());
     this.labels = [];
   }
   ```

---

## Step 3: Adding Voice Commands for Scene Management

1. **Create a New TypeScript Module for ASR**
   - Create `asrModule.ts`.
   - Import Snap’s automatic speech recognition (ASR) module.

2. **Implement Speech Event Handling**

   ```typescript
   onASRResult(eventArgs) {
     if (eventArgs.final && eventArgs.text.toLowerCase().includes('new')) {
       this.drawingComponent.newScene();
     }
   }
   ```

   > **Gotcha:** To allow inter-module dependencies, import and inject the DrawingComponent into the ASR module and link them in Lens Studio’s UI.

3. **Trigger Actions via Voice**
   - Now, saying “new scene” or any phrase containing “new” will reset your drawing.

---

## Step 4: Creating an ASCII Drawing Experience

1. **Set Up Drawing State in Code**
   - Add variables to track whether you’re currently drawing (based on pinch state).
   - Use an update event to create a new label at the hand’s position if drawing.

2. **Ensure Characters Don’t Overlap**
   - Check that the new character position isn’t too close to the last one (distance threshold).

3. **Randomize Characters**
   - Pick from an array of ASCII characters to create random “ASCII art.”

   ```typescript
   const asciiChars = ['@', '#', '$', '%', '&', '*', '+', '-', '='];
   const char = asciiChars[Math.floor(Math.random() * asciiChars.length)];
   ```

---

## Step 5: Setting Up SnapCloud (Supabase) for Real-Time Data and Persistence

### A. Create a SnapCloud Supabase Project

1. **Log into SnapCloud Supabase**
2. **Create a new project** (e.g., `ascii_drawing`) in the preferred region.

### B. Import Credentials into Lens Studio

- Use Lens Studio’s SnapCloud integration to add project credentials to your AR project.

### C. Add Supabase Client to Your Project

1. **Create a TypeScript module (`supabase.ts`):**

   ```typescript
   import { createClient } from '@supabase/snapcloud-js';

   export class SupabaseModule extends BaseScriptComponent {
     onAwake() {
       this.client = createClient('<SUPABASE_URL>', '<SUPABASE_ANON_KEY>');
     }
   }
   ```

2. **Authenticate and Prepare for Real-Time**
   - On instantiation, log in and confirm you’re authenticated.

   ```typescript
   this.client.auth.signIn();
   ```

   > **Gotcha:** The SnapCloud Supabase integration provides a real-time API and manages user authentication for each Lens app instance.

---

### D. Implement Real-Time Broadcasting of Drawing Events

1. **Create a Real-Time Channel:**

   ```typescript
   this.channel = this.client.channel('ascii_channel');
   this.channel.subscribe();
   ```

2. **Broadcast Character Data On Draw Event:**

   ```typescript
   this.channel.send({
     type: 'broadcast',
     event: 'new_character',
     payload: { char, x, y, z }
   });
   ```

---

### E. Create a Web Listener for Real-Time Updates

1. **Set Up a Basic Web App**
   - Install the Supabase JS client:

     ```bash
     npm install @supabase/supabase-js
     ```

   - Create and initialize a client:

     ```javascript
     const supabase = createClient('<SUPABASE_URL>', '<SUPABASE_ANON_KEY>');
     const channel = supabase.channel('ascii_channel');

     channel.on('broadcast', { event: 'new_character' }, (payload) => {
       // Add character to page at (x, y), adjust size based on z
       drawCharacter(payload.char, payload.x, payload.y, payload.z);
     });
     channel.subscribe();
     ```

2. **Display Characters in the DOM**
   - Adjust CSS position and size based on payload’s coordinates.

   > **Note:** For a richer experience, consider using a 3D web library like Three.js.

3. **Respond to “new_scene” Broadcasts**
   - Clear DOM when a “new_scene” event is received.

---

### F. Persisting Drawings in the Database

1. **Create Database Tables in SnapCloud:**

   ```sql
   -- Drawings table
   CREATE TABLE drawings (
     id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
     user_id uuid NOT NULL REFERENCES auth.users(id)
   );

   -- Positions table
   CREATE TABLE positions (
     id serial PRIMARY KEY,
     drawing_id uuid REFERENCES drawings(id),
     character text NOT NULL,
     x float8 NOT NULL,
     y float8 NOT NULL,
     z float8 NOT NULL
   );
   ```

2. **Enable Row Level Security (RLS):**
   - Only allow users to insert/select their own drawings:

     - For `drawings`: owner can read/insert/delete own records.
     - For `positions`: users can only manipulate records for their drawings.

3. **Import Database Types in TypeScript**

   ```typescript
   import type { Database } from './<generated-types>';
   ```

4. **On First Draw, Insert a Drawing:**

   ```typescript
   if (!this.currentDrawing) {
     const { data } = await this.client.from('drawings').insert([{
       user_id: this.client.auth.user().id
     }]).select().single();
     this.currentDrawing = data;
   }
   ```

5. **Insert Positions on Each Draw:**

   ```typescript
   await this.client.from('positions').insert([
     { drawing_id: this.currentDrawing.id, character: char, x, y, z }
   ]);
   ```

---

### G. Generating Images with Edge Functions and Storage

1. **Create a Storage Bucket called `images`**

2. **Write an Edge Function (in Supabase or SnapCloud)**
   - Receives a `drawing_id`.
   - Fetches all positions for that drawing.
   - Generates and stores an SVG image in the bucket.

3. **Trigger the Edge Function from Lens App**
   - Add a voice command (e.g., say “generate”) to invoke image generation.

   ```typescript
   if (eventArgs.text.toLowerCase().includes('generate')) {
     await this.supabase.createImage(this.currentDrawing.id);
   }
   ```

---

## Conclusion & Next Steps

Congrats, you’ve built an interactive, real-time AR drawing app for Snap Spectacles that syncs with the web and persists data with SnapCloud (Supabase). Here’s what you can explore next:

- Enhance the web in-browser rendering (3D models, animations)
- Add user galleries, sharing, or collaborative multi-user drawing
- Leverage more SnapCloud Edge Functions (generate GIFs, run ML models, etc.)
- Experiment with additional Spectacles modules: geolocation, LLM integration, etc.

**Where will AR web apps go from here? That’s up to you—and your next idea.**

---

> **Gotchas & Tips:**
>
> - Make sure all SnapCloud project credentials and real-time channel names match across Lens Studio and web.
> - Keep RLS policies strict—only the drawing owner should access their data.
> - Edge Functions give a lot of power—keep user input validation in mind!
> - SnapCloud Supabase dashboards mirror standard Supabase, but project provisioning and access go through Snap’s portal.