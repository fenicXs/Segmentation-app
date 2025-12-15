# Chest X-ray Segmentation Web App (n8n + Swin-UNet)

A lightweight frontend that sends chest X-ray images to an n8n workflow. The workflow calls your local Swin-UNet segmentation (via `n8n_inference.py`) and returns a colored overlay. Drop an image in the browser, get back the segmented overlay.

## Preview
![Upload screen](overview/Initial%20upload%20page.png)
![Processing](overview/N8N%20processing.png)
![Result](overview/segmentation%20result.png)

## Repo layout
- `webapp/` - static HTML/CSS/JS frontend (drag-and-drop, endpoint selector, overlay preview).
- `overview/` - screenshots used above.
- Backend/model lives outside this repo (`segmentation model/SwinSegmentation` in your workspace) and is invoked by n8n.

## Prerequisites
- A running n8n instance with the workflow from this project:
  - Webhook node path: `seg-chestxray`
  - Binary field: `file`
  - Execute Command node calls: `python n8n_inference.py --input ... --output-overlay ...`
- Model weights accessible to that n8n host (e.g., `best_model.pth`).
- For local hosting of the frontend: Python 3 (or any static file server).

## Run the frontend locally
```bash
cd webapp
python -m http.server 8080
# open http://localhost:8080
```
In the page, set the endpoint field to:
- Live (activated workflow): `http://localhost:5678/webhook/seg-chestxray`
- Test (not activated): `http://localhost:5678/webhook-test/seg-chestxray`

## Calling the workflow directly (curl)
```bash
curl -X POST "http://localhost:5678/webhook-test/seg-chestxray" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@C:/path/to/chest_xray.png" \
  --output segmented_overlay.png
```

## Deploy options
1) **GitHub Pages / Netlify / Vercel (static hosting)**  
   - Publish the `webapp/` folder.  
   - Update the endpoint in the page to your public n8n URL (HTTPS).

2) **Reverse proxy for n8n** (if self-hosted)  
   - Put n8n behind NGINX/Caddy with TLS.  
   - Expose `/webhook/seg-chestxray` over HTTPS so the browser can reach it.

3) **Optional proxy API**  
   - Front the webhook with a tiny FastAPI/Express service at `/segment` that forwards the file to n8n and returns the binary.  
   - Add auth/rate limits there if exposing publicly.

## Notes
- Ensure CORS is allowed if you host the frontend on a different origin than n8n (or use a proxy).
- Keep the `file` field name unchanged unless you also adjust the Webhook node.
- The overlay returned is a PNG; the mask is also written on disk by the inference script if you need it.
