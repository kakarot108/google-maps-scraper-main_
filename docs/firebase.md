# Firebase + Cloud Run Deployment

Firebase Hosting can sit in front of this project, but the Go application itself should run on Cloud Run. The simplest setup is:

- Firebase Hosting for the public entry point
- Cloud Run for the Go API/admin server
- A second Cloud Run service for the background worker if you want queue processing online

## What To Deploy

For the web/API server, deploy the SaaS entrypoint:

```bash
gcloud run deploy google-maps-scraper-web \
  --source . \
  --region us-central1 \
  --allow-unauthenticated \
  --command /app/gmapssaas \
  --args serve \
  --set-env-vars DATABASE_URL="postgres://USER:PASSWORD@HOST:5432/DB?sslmode=disable",ENCRYPTION_KEY="YOUR_32_BYTE_HEX_KEY"
```

If you also need the worker online, deploy a second service from the same image and change the command to `worker`:

```bash
gcloud run deploy google-maps-scraper-worker \
  --source . \
  --region us-central1 \
  --no-allow-unauthenticated \
  --command /app/gmapssaas \
  --args worker \
  --set-env-vars DATABASE_URL="postgres://USER:PASSWORD@HOST:5432/DB?sslmode=disable"
```

## Firebase Hosting

The repo includes a starter [firebase.json](../firebase.json) that rewrites all requests to the Cloud Run service named `google-maps-scraper-web` in `us-central1`.

Before deploying, update the service name or region if your Cloud Run setup uses different values.

Then connect Firebase to your project and deploy hosting:

```bash
firebase login
firebase use --add
firebase deploy --only hosting
```

## Notes

- Cloud Run needs the Go server to listen on port `8080`, which this repo already does.
- Firebase Hosting only fronts the HTTP entrypoint. It does not replace the worker process.
- If you only want the browser UI online, the same Cloud Run service can handle the UI and API together.