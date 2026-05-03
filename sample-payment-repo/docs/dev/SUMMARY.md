# SUMMARY.md

_Automatic documentation generation failed._

## Provider Status
Total providers: 3
Healthy providers: 1
- ✅ Gemini (errors: 0)
- ❌ Groq (errors: 3)
  - Last error: Error code: 429 - {'error': {'message': 'Rate limit reached for model `llama-3.3-70b-versatile` in organization `org_01k5mme46vfndbaxpmvvyx5fdf` service tier `on_demand` on tokens per day (TPD): Limit 100000, Used 98105, Requested 4591. Please try again in 38m49.344s. Need more tokens? Upgrade to Dev Tier today at https://console.groq.com/settings/billing', 'type': 'tokens', 'code': 'rate_limit_exceeded'}}
- ❌ DeepSeek (errors: 3)
  - Last error: Error code: 402 - {'error': {'message': 'Insufficient Balance', 'type': 'unknown_error', 'param': None, 'code': 'invalid_request_error'}}

## Code (truncated)
```
You are DocAI, an expert technical writer analyzing THIS SPECIFIC REPOSITORY.

Persona: dev
Repository: docai_manual_y1c1ai9h

ACTUAL CODEBASE ANALYSIS:
{
  "project_name": "docai_manual_y1c1ai9h",
  "file_count": 195,
  "languages": [
    "py",
    "ts",
    "js",
    "java",
    "rb"
  ],
  "main_directories": [
    ".",
    "./docs",
    "./.devcontainer",
    "./.devcontainer/custom-payment-flow-server-go",
    "./.devcontainer/payment-element-server-go",
    "./.devcontainer/prebuilt-checkout-page-server-python",
    "./.devcontainer/payment-element-server-python",
    "./.devcontainer/payment-element-server-java",
    "./.devcontainer/prebuilt-checkout-page-server-ruby",
    "./.devcontainer/custom-payment-flow-server-ruby",
    "./.devcontainer/payment-element-client-vue-cva",
    "./.devcontainer/prebuilt-checkout-page-server-node",
    "./.devcontainer/custom-payment-flow-server-dotnet",
    "./.devcontainer/custom-payment-flow-server-node",
    "./.devcontainer/payment-elemen...
```

## Manual Documentation Needed
Please add proper documentation manually.
