# Campaign Concept Studio

A production-oriented full-stack campaign concept workspace built with Next.js, TypeScript, and the OpenAI API.

## What it does

A marketer enters a campaign brief, audience, product details, tone, and channels. The app then generates:

- a concise campaign concept
- three distinct headline/body copy routes
- an ordered launch checklist
- three image prompts
- one lead campaign visual generated from the first image prompt

## Architecture

```text
Browser (Next.js React UI)
        |
        | POST /api/campaign
        v
Next.js route handler (server-only)
        |
        +--> OpenAI Responses API  -> structured campaign JSON
        |
        +--> OpenAI Images API     -> generated lead visual
        v
JSON response to browser
```

### Client / server boundary

`app/page.tsx` is client-side UI only. It never imports the OpenAI SDK, never reads `OPENAI_API_KEY`, and never calls `api.openai.com` directly.

`app/api/campaign/route.ts` is the server-side integration boundary. It validates input, calls OpenAI, and returns only the data needed by the UI. Keep `OPENAI_API_KEY` server-side and do not prefix it with `NEXT_PUBLIC_`.

## Local setup

1. Install Node.js 20+.
2. Copy `.env.example` to `.env.local`.
3. Set `OPENAI_API_KEY`.
4. Optionally tune `OPENAI_TEXT_MODEL`, `OPENAI_IMAGE_MODEL`, `OPENAI_IMAGE_QUALITY`, and `OPENAI_IMAGE_SIZE`.
5. Run:

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## Production

```bash
npm install
npm run build
npm start
```

The app can be deployed to any Node-capable Next.js host. For Vercel, add the environment variables in the project settings and deploy the repo.

## Model and prompt tuning

The current defaults are intentionally environment-driven so you can change models without touching the UI. The text generation prompt and JSON schema live in `lib/campaign.ts`. The image model, size, and quality are read from environment variables in `app/api/campaign/route.ts`.

For higher reasoning quality, increase `OPENAI_TEXT_MODEL` to a stronger current model. For faster, lower-cost image generation, keep a fast GPT-Image model; for higher-fidelity creative work, choose the stronger image model available to your account. Verify model availability and pricing in OpenAI's current model catalog before changing production defaults.

## Validation plan

### Automated / pre-release

- TypeScript compile and Next.js production build
- API input validation for empty or malformed fields
- No API key exposed through client bundles
- Successful structured JSON parsing
- Image generation failure returns a user-visible error rather than fake success
- Mobile and desktop responsive layout smoke test

### Manual acceptance test

1. Start the app with a valid `OPENAI_API_KEY`.
2. Submit the prefilled brief.
3. Confirm the campaign board contains one concept, exactly three copy variants, a checklist, exactly three image prompts, and a generated lead visual.
4. Turn off/omit the API key and verify the UI shows a configuration error rather than mock content.
5. Submit with missing inputs and verify a 400 validation response.
6. Confirm the browser never contains the OpenAI API key.

## Important production notes

This project deliberately does not fake generation, image creation, or configuration. If OpenAI is unavailable or credentials are missing, the UI reports the failure.

For a larger production system, add authentication, persistent campaign storage, usage/rate limits, observability, request tracing, moderation policies, and a durable asset store/CDN rather than returning base64 images directly in API responses.

## Reference docs

- https://developers.openai.com/api/docs/models
- https://developers.openai.com/api/reference/typescript/resources/beta/subresources/responses/methods/create
