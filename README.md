# Deploy and Host OpenMed on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/openmed-api?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=openmed-api)

[OpenMed](https://github.com/maziyarpanahi/openmed) is an open-source toolkit for clinical text: it finds diseases, drugs, genes and anatomy in medical notes, and it detects and removes personal information (names, dates of birth, record numbers, phones, addresses) in many languages. Its models are small open models that run on CPU. This template runs OpenMed's REST API from a pinned release, protected by an API key from the first boot, with downloaded models kept on a volume.

## About Hosting OpenMed

One service, the official OpenMed image, with a volume for the model cache. Models download from Hugging Face the first time a request uses them (a few seconds), then load from the volume after every redeploy.

OpenMed's own API key auth is turned on: every model route needs the `X-API-Key` header, while `/health`, `/readyz` and the `/docs` page stay open. The key is generated when you deploy. Request logs never contain the text you send.

## Common Use Cases

- De-identifying clinical notes before they reach an LLM, a data warehouse or a vendor
- Extracting diagnoses, medications and other entities from free-text medical records
- A PII-redaction API for support tickets, transcripts or forms, in English and other languages
- A self-hosted, open alternative to cloud medical NLP services, with data that stays in your project

## Dependencies for OpenMed Hosting

- Nothing external. Models come from Hugging Face; set `HF_TOKEN` only for gated models or heavy use.

### Deployment Dependencies

- [OpenMed on GitHub](https://github.com/maziyarpanahi/openmed)
- [REST service guide](https://github.com/maziyarpanahi/openmed/blob/master/docs/rest-service.md)
- [OpenMed models on Hugging Face](https://huggingface.co/OpenMed)

### Implementation Details

**First use:** copy `OPENMED_API_KEY` from the service's Variables tab, then:

```bash
curl -X POST https://<your-domain>/pii/deidentify \
  -H "X-API-Key: $OPENMED_API_KEY" -H "Content-Type: application/json" \
  -d '{"text":"Patient John Smith, DOB 03/14/1962, phone 555-201-3344.","lang":"en","method":"mask"}'
# "Patient [first_name] [last_name], DOB [date_of_birth], phone [phone_number]."
```

Medical entities:

```bash
curl -X POST https://<your-domain>/analyze \
  -H "X-API-Key: $OPENMED_API_KEY" -H "Content-Type: application/json" \
  -d '{"text":"Type 2 diabetes and hypertension, started on metformin.","model_name":"disease_detection_superclinical"}'
```

Open `https://<your-domain>/docs` for every endpoint: `/pii/extract`, `/pii/deidentify` (mask, remove, replace or hash), `/analyze`, `/brief`, `/ground`, streaming and async jobs.

**Models and memory:** idle uses about 200 MB. The default English PII model brings it to about 350 MB; a large entity model (434M parameters) adds about 2 GB, so give the service 2.5 GB or more for those. `OPENMED_SERVICE_KEEP_ALIVE` unloads idle models, and `OPENMED_SERVICE_MAX_RESIDENT_MODELS` caps how many stay loaded. List models in `OPENMED_SERVICE_PRELOAD_MODELS` to load them at boot, so the first request isn't slow.

**Speed:** `OMP_NUM_THREADS` caps CPU threads per call. Without it torch starts one thread per host core, and a short request takes seconds instead of a quarter of one.

**Custom domain:** add it to `OPENMED_SERVICE_TRUSTED_HOSTS`, or OpenMed rejects requests to it.

Notes and limits:

- Change `OPENMED_API_KEY` and redeploy to rotate the key.
- OpenMed is a research toolkit, not a certified medical device. Review de-identified output before treating it as anonymous.
- Each template update pins a newer OpenMed release.

## Why Deploy OpenMed on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying OpenMed on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
