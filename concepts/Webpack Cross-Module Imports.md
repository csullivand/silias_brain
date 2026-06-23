---
tags: [concept, webpack, silia, lambda]
---

# Webpack Cross-Module Imports — encoder.json Pattern

When a Lambda handler imports a model from another module (e.g., DynamicTables importing `ChatbotModel` from Assistant), webpack bundles the FULL dependency tree.

## The Problem

If any dependency reads files from disk at startup (like the OpenAI tokenizer → `encoder.json`, `vocab.bpe`), those files must be in the Lambda bundle. But webpack's `CopyWebpackPlugin` via `findModelFiles()` only searches **within the current module's** `application/` directory.

## The Fix

Copy the required data files (`encoder.json`, `vocab.bpe`) into the importing module's `application/` directory. The webpack config at `webpack.config.js:52-91` will find and bundle them.

## Pattern

```
# Check if your module imports ChatbotModel (or any model with heavy deps)
grep -r 'import.*ChatbotModel' YourModule/application/

# If yes, copy the tokenizer files
cp Assistant/application/SignalWebhook/encoder.json YourModule/application/
cp Assistant/application/SignalWebhook/vocab.bpe YourModule/application/
```

## Modules That Need It

Any module importing `ChatbotModel` from Assistant: Billing, DynamicTables, Flows, Folders, GeneratorAI, Integrations, Messages, Metrics, Training, Workflows.

## Related
- [[Dynamic Tables Refactor Plan]]
- webpack.config.js lines 52-91
