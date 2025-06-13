## local development and testing

### Setup a venv

```
python3.11 -m venv venv
source venv/bin/activate
```

### Install dependencies

```
pip install fastmcp==2.2.2
```

### Run MCP Server

Note: runs on 8000 by default

```
python weather.py
```

### Review Tools

First check to see which tools are already registered

```
LLAMA_STACK_ENDPOINT=http://localhost:8321
curl -sS $LLAMA_STACK_ENDPOINT/v1/toolgroups -H "Content-Type: application/json" | jq
```

### Register the weather MCP server

If running Llama Stack in a container

```
curl -X POST -H "Content-Type: application/json" --data '{ "provider_id" : "model-context-protocol", "toolgroup_id" : "mcp::weather", "mcp_endpoint" : { "uri" : "http://host.docker.internal:8000/sse"}}' $LLAMA_STACK_ENDPOINT/v1/toolgroups
```

Else 

```
curl -X POST -H "Content-Type: application/json" --data '{ "provider_id" : "model-context-protocol", "toolgroup_id" : "mcp::weather", "mcp_endpoint" : { "uri" : "http://localhost:8000/sse"}}' $LLAMA_STACK_ENDPOINT/v1/toolgroups
```

### Check registration 

```
curl -sS $LLAMA_STACK_ENDPOINT/v1/toolgroups -H "Content-Type: application/json" | jq
```

Register the 8B model for better tool calling

```
curl -sS $LLAMA_STACK_ENDPOINT/v1/models -H "Content-Type: application/json" | jq -r '.data[].identifier'
```

### Test connectivity

```
export LLAMA_STACK_MODEL=meta-llama/Llama-3.2-3B-Instruct
or 
export LLAMA_STACK_MODEL=meta-llama/Llama-3.1-8B-Instruct
```

```
API_KEY=none
LLAMA_STACK_ENDPOINT=http://localhost:8321

curl -sS $LLAMA_STACK_ENDPOINT/v1/inference/chat-completion \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_KEY" \
  -d "{
     \"model_id\": \"$LLAMA_STACK_MODEL\",
     \"messages\": [{\"role\": \"user\", \"content\": \"what model are you?\"}],
     \"temperature\": 0.0
   }" | jq -r '.completion_message | select(.role == "assistant") | .content'
```

And test the weather tool invocation via a Llama Stack Client

```
python test-weather.py
```



