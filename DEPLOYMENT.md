# Streamlit Cloud Deployment Guide

## Important: Model Files & Streamlit Cloud

Your app uses large model files that **cannot be committed to GitHub** (they exceed GitHub's 100MB file limit). When deploying to Streamlit Cloud, you have several options:

### Option 1: Manual Model Upload in App (Recommended for testing)
1. Deploy the app to Streamlit Cloud (as-is, without model files)
2. Once deployed, use the sidebar "Upload GAN checkpoint" feature to upload `progan_generator_final_2.pt`
3. The uploaded file is stored in Streamlit Cloud's temporary storage (persists during session)

**Limitations:** Models must be re-uploaded after app redeploy or session restart.

### Option 2: Upload Models to Cloud Storage (Recommended for production)
1. Upload model files to Google Drive, Dropbox, AWS S3, or similar
2. Modify `app.py` to download models on startup:

```python
import os
import requests

# Example: Download from Google Drive
def download_model_if_missing():
    model_path = "models/progan_generator_final_2.pt"
    if not os.path.exists(model_path):
        # Replace with your actual download URL
        url = "https://drive.google.com/uc?id=YOUR_FILE_ID&export=download"
        response = requests.get(url)
        with open(model_path, 'wb') as f:
            f.write(response.content)
```

3. Call this function on app startup

### Option 3: Use Streamlit Secrets for DropBox/Drive Links
1. In Streamlit Cloud dashboard, set a secret `MODEL_DOWNLOAD_URL`
2. Modify app.py to download from that URL on startup

### Option 4: Reduce Model Size
- Train smaller versions of your models
- Use model quantization/pruning
- Use lightweight architectures

## Steps to Deploy to Streamlit Cloud

1. **Push this repository to GitHub** (see instructions below)

2. **Create Streamlit Cloud account**
   - Go to https://streamlit.io/cloud
   - Sign in with GitHub

3. **Deploy**
   - Click "New app"
   - Select this repository
   - Choose main branch and set main file to `app.py`
   - Click Deploy

4. **Handle Model Files**
   - Choose one of the options above
   - Test the app

5. **Manage Secrets** (if using Option 3/4)
   - In app settings, add secrets for model URLs
   - Access via `st.secrets["MODEL_DOWNLOAD_URL"]`

## Troubleshooting

**"Load checkpoint models first" error in Streamlit Cloud**
- Models haven't been uploaded or downloaded yet
- Use the sidebar to upload models, or implement auto-download

**App stops after 1 hour of inactivity**
- Streamlit Cloud stops inactive apps
- Just refresh the URL to restart

**Module not found errors**
- Ensure all dependencies in `requirements.txt` are Python packages
- PyTorch must be installed with correct CUDA version

