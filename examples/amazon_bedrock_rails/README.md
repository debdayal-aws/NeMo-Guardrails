# Bedrock Integration

This integration uses the below components for NeMo Guardrail integration -
- Embedding Model: Amazon Bedrock Titan
- Embedding Store: In-memory Spotify Annoy lib, this can be changed with other embeddind stores if needed following NeMo configurations
- LLM: Amazon Bedrock - Cohere Command model

Below environment varliable need to be set for accessing Bedrock:

```
BEDROCK_REGION=us-east-1
BEDROCK_ENDPOINT_URL=https://bedrock-runtime.us-east-1.amazonaws.com

```


Rest of the configuration is same as standard NeMo Guardrails configurations and are not repeated here.

## Working with Streaming output

NeMo Guardrails does not support streaming output at this time, however if you do not have output moderation, streaming can be supported with a "hack" using Langchain or similar libs. Instead of handing over the LLM Langchain to NeMo write a action handler as below:

```
async def pass_to_llm_chain(input):
    """
    This is an action that delegates to the LLM Chain.
    Args:
        input: Users original prompt or a modified prompt by NeMo.
    Returns:
        Users original prompt or a modified prompt by NeMo
    """
    return f"PASS_TO_LLM_CHAIN:{input}"

# Create rails 
rails = LLMRails(config=*nemo_config_dir*, verbose=True)
rails.register_action(pass_to_llm_chain, "pass_to_llm_chain")

```

In the colong file use the action handler like any other action handler.
**
```

define flow
    user ...
    $answer = execute pass_to_llm_chain(input=$last_user_message)
    bot $answer
```

Now, in the application side, inspect the NeMo output, if the output starts with "PASS_TO_LLM_CHAIN:" then extract the prompt from the string and use regular LLM chain with straming support!

