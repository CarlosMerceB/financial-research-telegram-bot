# How to Export Your n8n Workflow

Follow these steps to export the workflow JSON and add it to this repository.

## Export from n8n

1. Open your **Financial Research Bot** workflow in n8n
2. Click the **⋮** (three dots menu) in the top-right corner
3. Select **Download**
4. n8n will download a `.json` file — this is your complete workflow

## Add to Repository

1. Rename the downloaded file to `workflow.json`
2. Move it into this repository's root folder
3. Commit and push:
   ```bash
   git add workflow.json
   git commit -m "Add n8n workflow export"
   git push
   ```

## Security Note

The exported JSON may contain your API keys in plain text inside node configurations. Before committing:

1. Open `workflow.json` in a text editor
2. Search for and replace:
   - Your **Anthropic API key** → replace with `YOUR_ANTHROPIC_API_KEY`
   - Your **html2pdf.app API key** → replace with `YOUR_HTML2PDF_API_KEY`
   - Your **Telegram Bot Token** → replace with `YOUR_TELEGRAM_BOT_TOKEN`
3. Google Drive credentials are stored separately in n8n's credential system and are NOT included in the export

**Important:** Double-check the file before pushing. Run this search to make sure no keys remain:
```bash
# Look for anything that looks like an API key
grep -i "apikey\|api_key\|x-api-key\|token\|Bearer" workflow.json
```
