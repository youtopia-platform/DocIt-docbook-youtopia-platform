# api.md

_Automatic documentation generation failed._

## Provider Status
Total providers: 1
Healthy providers: 0
- ❌ Gemini (errors: 3)
  - Last error: 429 You exceeded your current quota, please check your plan and billing details. For more information on this error, head to: https://ai.google.dev/gemini-api/docs/rate-limits. To monitor your current usage, head to: https://ai.dev/rate-limit. 
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_requests, limit: 0, model: gemini-2.0-flash
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_requests, limit: 0, model: gemini-2.0-flash
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_input_token_count, limit: 0, model: gemini-2.0-flash
Please retry in 23.968709429s. [links {
  description: "Learn more about Gemini API quotas"
  url: "https://ai.google.dev/gemini-api/docs/rate-limits"
}
, violations {
  quota_metric: "generativelanguage.googleapis.com/generate_content_free_tier_requests"
  quota_id: "GenerateRequestsPerDayPerProjectPerModel-FreeTier"
  quota_dimensions {
    key: "model"
    value: "gemini-2.0-flash"
  }
  quota_dimensions {
    key: "location"
    value: "global"
  }
}
violations {
  quota_metric: "generativelanguage.googleapis.com/generate_content_free_tier_requests"
  quota_id: "GenerateRequestsPerMinutePerProjectPerModel-FreeTier"
  quota_dimensions {
    key: "model"
    value: "gemini-2.0-flash"
  }
  quota_dimensions {
    key: "location"
    value: "global"
  }
}
violations {
  quota_metric: "generativelanguage.googleapis.com/generate_content_free_tier_input_token_count"
  quota_id: "GenerateContentInputTokensPerModelPerMinute-FreeTier"
  quota_dimensions {
    key: "model"
    value: "gemini-2.0-flash"
  }
  quota_dimensions {
    key: "location"
    value: "global"
  }
}
, retry_delay {
  seconds: 23
}
]

## Code (truncated)
```
You are DocAI, an expert API writer documenting REAL endpoints from this repository.

Persona: dev
Repository: docai_manual_3cuu65vv
CODE ANALYSIS:
{
  "project_name": "docai_manual_3cuu65vv",
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
    "./.devcontainer/payment-element-cli...
```

## Manual Documentation Needed
Please add proper documentation manually.
