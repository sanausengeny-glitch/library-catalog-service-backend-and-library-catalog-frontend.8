# Deployment

## Status

The app is prepared for Render (backend) and Vercel (frontend), with MongoDB Atlas as the database. It has not yet been published: this environment has no Render/Vercel deployment CLI or configured production MongoDB connection. Consequently, live URLs are not available yet.

- Frontend: pending Vercel deployment
- Backend: pending Render deployment
- Repository: [library-catalog-service](https://github.com/sanausengeny-glitch/library-catalog-service-backend-and-library-catalog-frontend.8)

The repository currently contains the frontend under `client/`; it is not a second local Git repository.

## Required environment variables

Set these in the hosting provider dashboards. Do not commit production values.

| Provider | Variable | Value |
| --- | --- | --- |
| Render | `MONGODB_URI` | MongoDB Atlas connection string for the production database |
| Render | `JWT_SECRET` | A long, unique secret; the Render blueprint generates one |
| Render | `FRONTEND_ORIGIN` | Exact deployed frontend origin, e.g. `https://your-app.vercel.app` |
| Render | `NODE_ENV` | `production` (set by `render.yaml`) |
| Vercel | `VITE_API_URL` | Render backend origin, e.g. `https://your-api.onrender.com`, without a trailing slash |

Render supplies `PORT`. The backend refuses to start in production without `MONGODB_URI` or `JWT_SECRET`.

## Deployment steps

1. Create a MongoDB Atlas database and copy its connection string into Render's `MONGODB_URI` setting.
2. Create a Render Web Service from this repository and apply `render.yaml`. The service root is the repository root; its health check is `/health`.
3. Create a Vercel project from the same repository with the project root set to `client`. Set `VITE_API_URL` to the Render service origin and deploy.
4. Set Render's `FRONTEND_ORIGIN` to the deployed Vercel origin and redeploy the backend. The API only grants CORS access to that exact origin.
5. Verify the backend's `GET /genres` response contains data, then open the frontend and test a catalog search. Confirm the browser requests go to the Render origin, not `localhost`.

`render.yaml` and `client/vercel.json` provide the service health check and frontend history fallback. For local setup, use `.env.example` and `client/.env.example`; the Vite development proxy works when `VITE_API_URL` is empty.

## Deployment issue found and resolved

During production-readiness review, the frontend's API calls were relative URLs and depended on Vite's development-only proxy. In a deployed frontend, those requests would go to the Vercel origin rather than the backend. The frontend now prefixes API requests with `VITE_API_URL`, and the backend allows the configured `FRONTEND_ORIGIN` through CORS. This was caught before publishing; no hosted deployment has yet occurred, so the final live URLs and any issues encountered during an actual provider rollout still need to be recorded here.
