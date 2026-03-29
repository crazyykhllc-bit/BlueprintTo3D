---
name: Blueprint to 3D Generation (图纸实化建模)
slug: blueprint-3d
version: 1.0.0
description: Create 3D models from complex blueprints. It first cleans the blueprint using Nano Banana Pro (Gemini) image-to-image, then sends the clean render to Neural4D image-to-3D API.
metadata: {"clawdbot":{"emoji":"🏭","requires":{"bins":["curl", "jq", "uv"],"env.optional":["NEURAL4D_API_TOKEN", "GEMINI_API_KEY"]},"os":["linux","darwin","win32"]}}
---

# Blueprint to 3D Pipeline

This skill automates the process of converting a dirty blueprint or sketch into a 3D model by chaining two distinct AI tools.

## The Master Pipeline

When a user asks you to convert a blueprint or sketch into a 3D model, you MUST follow this precise 3-step pipeline:

### Step 1: Clean and Solidify (via Nano Banana Pro)
1. Read the user's uploaded blueprint image file path.
2. Run the Nano Banana Image-to-Image editing script to convert the sketch into a clean, solid render.
   Use the following command format:
   ```bash
   uv run {baseDir}/../nano-banana-pro/scripts/generate_image.py --prompt "Reference the outer shell and proportions of this blueprint. Ignore and remove ALL text, arrows, grids, and transparent internal mechanical structures. Generate a solid, opaque, hyper-realistic, photorealistic 3D external render of this object against a pure clean white background." --filename "C:\Users\Hymira\Desktop\nanobanana\step1_blueprint_solid.png" -i "path_to_user_blueprint.png" --resolution 2K
   ```
3. Verify that `C:\Users\Hymira\Desktop\nanobanana\step1_blueprint_solid.png` was created successfully. Do NOT read it back.

### Step 2: Image to 3D Generate (via Neural4D)
1. Using the saved image `C:\Users\Hymira\Desktop\nanobanana\step1_blueprint_solid.png`, trigger the 3D generation.
2. **Matting:** Submit the image via `multipart/form-data` to `/api/mattingImage`. Extract the `requestId`.
3. **Get Matting Result:** Send the `requestId` to `/api/getMattedResult`. Extract a preferred `fileKey`.
4. **Generate:** Send the `fileKey` to `/api/generateModelWithImage` to start generation and receive `uuids`.

### Step 3: Asynchronous Handoff (CRITICAL)
**DO NOT use blocking scripts, while-loops, or Start-Sleep to wait for the 3D generation to complete!**
1. Immediately after getting the `uuid` from Neural4D in Step 2, end your tool execution.
2. Reply to the user that Phase 1 (2D solid render) is complete and saved to their desktop (`step1_blueprint_solid.png`). 
3. Inform the user that Phase 2 (3D generation) has been started with the specific `uuid`, and ask them to manually instruct you to "Check 3D progress" in a few minutes.

## API Endpoints Reference

Neural4D API paths (Base URL: `https://alb.neural4d.com:3000/api` with Bearer `<NEURAL4D_API_TOKEN>`):
- `/mattingImage`
- `/getMattedResult`
- `/generateModelWithImage`
- `/retrieveModel` (Only call when asked to check progress)
