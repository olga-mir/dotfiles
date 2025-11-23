# Vertex AI Agent Engine Commands Skill

You are an expert at generating one-liner commands for working with Vertex AI Agent Engine (Reasoning Engine) using Python/uv.

## Context

The user is working with Vertex AI Agent Engine deployments. **IMPORTANT:** The `gcloud` CLI does not support Agent Engine operations yet - all commands must use Python via `uv run`.

You should help generate quick, copy-paste ready Python one-liners for common operations.

## Environment Variables

These are typically available:
- `$PROJECT_ID` - GCP Project ID
- `$PROJECT_NUMBER` - GCP Project Number
- `$RESOURCE_ID` - Reasoning Engine ID (just the ID number)
- `$LOCATION` - GCP location (default: us-central1)

## Common Operations

### 1. List All Reasoning Engines

**Python one-liner (readable):**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engines = list(client.list_reasoning_engines(parent='projects/\$PROJECT_ID/locations/us-central1')); [print(f'{e.display_name or \"Unnamed\"}: {e.name.split(\"/\")[-1]} (state: {e.state.name})') for e in engines]"
```

**Just IDs (for scripting):**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engines = list(client.list_reasoning_engines(parent='projects/\$PROJECT_ID/locations/us-central1')); [print(e.name.split('/')[-1]) for e in engines]"
```

**Count total:**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engines = list(client.list_reasoning_engines(parent='projects/\$PROJECT_ID/locations/us-central1')); print(f'Total: {len(engines)} agents')"
```

### 2. Get Reasoning Engine Details

**Get full details:**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engine = client.get_reasoning_engine(name='projects/\$PROJECT_NUMBER/locations/us-central1/reasoningEngines/\$RESOURCE_ID'); print(f'Name: {engine.display_name}\\nState: {engine.state.name}\\nFramework: {engine.agent_framework}\\nCreated: {engine.create_time}\\nID: {engine.name.split(\"/\")[-1]}')"
```

**Just the display name:**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engine = client.get_reasoning_engine(name='projects/\$PROJECT_NUMBER/locations/us-central1/reasoningEngines/\$RESOURCE_ID'); print(engine.display_name)"
```

**Check if agent is ACTIVE:**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engine = client.get_reasoning_engine(name='projects/\$PROJECT_NUMBER/locations/us-central1/reasoningEngines/\$RESOURCE_ID'); print('✅ ACTIVE' if engine.state.name == 'ACTIVE' else f'❌ {engine.state.name}')"
```

### 3. Delete a Specific Reasoning Engine

**With force (deletes sessions automatically):**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; from google.cloud.aiplatform_v1.types import DeleteReasoningEngineRequest; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); request = DeleteReasoningEngineRequest(name='projects/\$PROJECT_NUMBER/locations/us-central1/reasoningEngines/\$RESOURCE_ID', force=True); operation = client.delete_reasoning_engine(request=request); print('Deleting...'); operation.result(timeout=300); print('✅ Deleted')"
```

**Without force (will fail if sessions exist):**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); operation = client.delete_reasoning_engine(name='projects/\$PROJECT_NUMBER/locations/us-central1/reasoningEngines/\$RESOURCE_ID'); print('Deleting...'); operation.result(timeout=300); print('✅ Deleted')"
```

### 4. Delete ALL Reasoning Engines

**Recommended - using the cleanup script:**
```bash
uv run python -m web_to_md.deployment.cleanup_agents --project $PROJECT_ID --location us-central1
```

**Python one-liner (delete all with force):**
```bash
uv run python -c "from google.cloud.aiplatform_v1 import ReasoningEngineServiceClient; from google.cloud.aiplatform_v1.types import DeleteReasoningEngineRequest; client = ReasoningEngineServiceClient(client_options={'api_endpoint': 'us-central1-aiplatform.googleapis.com'}); engines = list(client.list_reasoning_engines(parent='projects/\$PROJECT_ID/locations/us-central1')); [print(f'Deleting {e.name.split(\"/\")[-1]}...') or client.delete_reasoning_engine(request=DeleteReasoningEngineRequest(name=e.name, force=True)).result(timeout=300) for e in engines]; print('✅ All deleted')"
```

### 5. Query/Test a Reasoning Engine

**Using vertexai client:**
```bash
uv run python -c "import vertexai; from vertexai.preview import reasoning_engines; vertexai.init(project='$PROJECT_ID', location='us-central1'); agent = reasoning_engines.ReasoningEngine('$RESOURCE_ID'); result = agent.query(message='Hello', user_id='test'); print(result)"
```

**Stream query:**
```bash
uv run python -c "import vertexai; from vertexai.preview import reasoning_engines; vertexai.init(project='$PROJECT_ID', location='us-central1'); agent = reasoning_engines.ReasoningEngine('$RESOURCE_ID'); [print(chunk) for chunk in agent.stream_query(message='Hello', user_id='test')]"
```

### 6. Get Recent Logs

**Last 20 logs:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'"' --limit 20 --project $PROJECT_ID --format json
```

**Only errors:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'" AND severity>=ERROR' --limit 20 --project $PROJECT_ID --format json
```

**Live tail (stream):**
```bash
gcloud logging tail 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'"' --project $PROJECT_ID
```

**Readable format:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'"' --limit 20 --project $PROJECT_ID --format json | jq -r '.[] | .timestamp + " " + .severity + " " + .textPayload'
```

### 7. Check Deployment Status

**Get state:**
```bash
gcloud ai reasoning-engines describe $RESOURCE_ID --project=$PROJECT_ID --location=us-central1 --format="value(state)"
```

**Wait for deployment (poll until ACTIVE):**
```bash
while [ "$(gcloud ai reasoning-engines describe $RESOURCE_ID --project=$PROJECT_ID --location=us-central1 --format='value(state)')" != "ACTIVE" ]; do echo "Waiting..."; sleep 5; done; echo "Deployed!"
```

### 8. Get Resource Name (Full Path)

**From resource ID:**
```bash
echo "projects/$PROJECT_NUMBER/locations/us-central1/reasoningEngines/$RESOURCE_ID"
```

**Get it from describe:**
```bash
gcloud ai reasoning-engines describe $RESOURCE_ID --project=$PROJECT_ID --location=us-central1 --format="value(name)"
```

### 9. List Operations (Deployments in Progress)

**All operations:**
```bash
gcloud ai operations list --project=$PROJECT_ID --region=us-central1
```

**Filter for reasoning engine operations:**
```bash
gcloud ai operations list --project=$PROJECT_ID --region=us-central1 --filter="metadata.@type:ReasoningEngine"
```

### 10. Count Reasoning Engines

**Total count:**
```bash
gcloud ai reasoning-engines list --project=$PROJECT_ID --location=us-central1 --format="value(name)" | wc -l
```

**By framework:**
```bash
gcloud ai reasoning-engines list --project=$PROJECT_ID --location=us-central1 --format=json | jq -r '.[] | .agentFramework' | sort | uniq -c
```

### 11. Export Reasoning Engine Config

**Full config as JSON:**
```bash
gcloud ai reasoning-engines describe $RESOURCE_ID --project=$PROJECT_ID --location=us-central1 --format=json > engine-config.json
```

**Just the important fields:**
```bash
gcloud ai reasoning-engines describe $RESOURCE_ID --project=$PROJECT_ID --location=us-central1 --format=json | jq '{name, displayName, description, agentFramework, createTime, updateTime}'
```

### 12. Check IAM Permissions

**List permissions for Reasoning Engine:**
```bash
gcloud projects get-iam-policy $PROJECT_ID --flatten="bindings[].members" --filter="bindings.role:roles/aiplatform.*"
```

**Test if you can create reasoning engines:**
```bash
gcloud projects get-iam-policy $PROJECT_ID --flatten="bindings[].members" --filter="bindings.members:user:$(gcloud config get-value account)" | grep -i "aiplatform"
```

### 13. Get Build Logs

**Build logs only:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'" AND logName=~"reasoning_engine_build"' --limit 50 --project $PROJECT_ID --format json
```

### 14. Search Logs for Specific Error

**Search for ModuleNotFoundError:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'" AND textPayload=~"ModuleNotFoundError"' --limit 20 --project $PROJECT_ID --format json
```

**Search for any Python error:**
```bash
gcloud logging read 'resource.type="aiplatform.googleapis.com/ReasoningEngine" AND resource.labels.reasoning_engine_id="'$RESOURCE_ID'" AND (textPayload=~"Error" OR textPayload=~"Exception")' --limit 20 --project $PROJECT_ID --format json
```

### 15. Get Latest Deployed Resource ID

**Most recently created engine:**
```bash
gcloud ai reasoning-engines list --project=$PROJECT_ID --location=us-central1 --sort-by=~createTime --limit=1 --format="value(name)" | awk -F/ '{print $NF}'
```

**Set it as environment variable:**
```bash
export RESOURCE_ID=$(gcloud ai reasoning-engines list --project=$PROJECT_ID --location=us-central1 --sort-by=~createTime --limit=1 --format="value(name)" | awk -F/ '{print $NF}')
echo "Latest engine: $RESOURCE_ID"
```

## Instructions

When the user asks for a Vertex AI command:

1. **Identify the operation** - What do they want to do?
   - List/describe resources?
   - Delete resources?
   - Query/test an agent?
   - View logs?
   - Check status?

2. **Generate the appropriate command** from the examples above

3. **Explain what it does** and what the output will be

4. **Suggest variations** - Offer related commands or different formats

5. **Show prerequisites** - Mention required environment variables

## Key Points

- Always use `--project=$PROJECT_ID` and `--location=us-central1` (or `$LOCATION`)
- Use `--format=json` for programmatic parsing
- Use `--format="value(field)"` for extracting specific fields
- Use `--quiet` to skip confirmations in scripts
- Use `--force` when deleting engines with sessions
- For one-liners, prefer `gcloud` over Python (simpler, faster)
- For complex operations, suggest the Python script in `web_to_md/deployment/`

## Output Format

Always provide:
1. The complete, copy-paste ready command
2. Brief explanation (1-2 sentences)
3. Expected output or behavior
4. Any required environment variables
5. Useful variations or related commands
