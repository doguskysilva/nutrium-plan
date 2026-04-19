# nutrium-plan

Static JSON plan host for Nutrium.

## Files
- `plan.json` - nutrition plan contract consumed by the app
- `Dockerfile` - serves `plan.json` through nginx

## URL
Production URL:

`https://nutrium.dogusky.cloud/plan.json`

After deploy, this endpoint should return the current plan JSON used by the app.
