# workflow.md

_Automatic documentation generation failed._

## Provider Status
Total providers: 3
Healthy providers: 3
- ✅ Gemini (errors: 1)
  - Last error: 429 You exceeded your current quota, please check your plan and billing details. For more information on this error, head to: https://ai.google.dev/gemini-api/docs/rate-limits. To monitor your current usage, head to: https://ai.dev/rate-limit. 
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_requests, limit: 0, model: gemini-2.0-flash
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_requests, limit: 0, model: gemini-2.0-flash
* Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_input_token_count, limit: 0, model: gemini-2.0-flash
Please retry in 42.329117196s. [links {
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
  seconds: 42
}
]
- ✅ Groq (errors: 2)
  - Last error: Error code: 429 - {'error': {'message': 'Rate limit reached for model `llama-3.3-70b-versatile` in organization `org_01k5mme46vfndbaxpmvvyx5fdf` service tier `on_demand` on tokens per day (TPD): Limit 100000, Used 96991, Requested 4581. Please try again in 22m38.208s. Need more tokens? Upgrade to Dev Tier today at https://console.groq.com/settings/billing', 'type': 'tokens', 'code': 'rate_limit_exceeded'}}
- ✅ DeepSeek (errors: 2)
  - Last error: Error code: 402 - {'error': {'message': 'Insufficient Balance', 'type': 'unknown_error', 'param': None, 'code': 'invalid_request_error'}}

## Code (truncated)
```
You are DocAI, an expert process analyst documenting ACTUAL workflows.

Persona: dev
Repository: docai_manual_3yid8vr1
VERSION: v1.0
CODE ANALYSIS:
{
  "project_name": "docai_manual_3yid8vr1",
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
