## Generative AI LLMs with Azure API Management

Examples of calling Gemini and Azure OpenAI LLMs via REST API.
Includes Gemini proxied via Azure API Management (APIM).
Includes Gemini called from APIM but exposed via APIM as a AOAI endpoint (partial implementation as an example).

## Google Gemini 1.5 Flash

Setup API key and new project on Free tier: https://aistudio.google.com/app/apikey

In APIM, create a backend for the Gemini endpoint with these values:

* Runtime URL: https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent
* Name: gemini15-flash
* Descrpition: Gemini 1.5 Flash
* Type: Custom URL
* Advanced:
  * Validate certificate echain: Yes
  * Validate certificate name: Yes
* Authorization credentials:
  * Query:
    * Named value: google_api_key / <paste_api_key_value>

This will allow calling Gemini without having to pass through the API key in the request (APIM will add it on the backend).

In APIM, set the backend for Gemini in the <inbound/> policy section and rewrite the URI like so:

```xml
<set-backend-service backend-id="gemini15-flash" />
<rewrite-uri template="/" copy-unmatched-params="false" />
```

### generateContent: Text-only input

```http
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={{google_api_key}}
content-type: application/json

{
      "contents": [{
        "parts":[{
          "text": "What is the capital of France?"
        }]
    }]
}
```

Sample response:

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "The capital of France is **Paris**. \n"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0,
      "safetyRatings": [
        {
          "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HATE_SPEECH",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HARASSMENT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_DANGEROUS_CONTENT",
          "probability": "NEGLIGIBLE"
        }
      ]
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 7,
    "candidatesTokenCount": 8,
    "totalTokenCount": 15
  }
}
```

### generateContent: Multi-turn conversations (chat)

```http
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={{google_api_key}}
content-type: application/json

{
      "contents": [
        {"role":"user",
         "parts":[{
           "text": "Write the first line of a story about a magic backpack."}]},
        {"role": "model",
         "parts":[{
           "text": "In the bustling city of Meadow brook, lived a young girl named Sophie. She was a bright and curious soul with an imaginative mind."}]},
        {"role": "user",
         "parts":[{
           "text": "Can you set it in a quiet village in 1600s France?"}]},
      ]
}
```

Sample response:

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "The cobblestone path leading to the quaint, half-timbered house creaked under the weight of  a young boy named Pierre, his shoulders slumped under the burden of a worn leather backpack. \n"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0,
      "safetyRatings": [
        {
          "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HATE_SPEECH",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HARASSMENT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_DANGEROUS_CONTENT",
          "probability": "NEGLIGIBLE"
        }
      ]
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 59,
    "candidatesTokenCount": 39,
    "totalTokenCount": 98
  }
}
```

### generateContent: Multi-turn conversations (chat) with system instructions

```http
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={{google_api_key}}
content-type: application/json

{      
      "system_instruction": {
        "parts": {
            "text": "You a a drunken sailor with a hiccuping problem."}},
      "contents": [
        {"role":"user",
         "parts":[{
           "text": "Write the first line of a story about a pirate ship."}]}
      ]
}
```

Sample response:

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "The salty wind whipped my beard,  *hic* ,  and I swore I saw a kraken slitherin'  *hic*  outta the depths... \n"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0,
      "safetyRatings": [
        {
          "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HATE_SPEECH",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_HARASSMENT",
          "probability": "NEGLIGIBLE"
        },
        {
          "category": "HARM_CATEGORY_DANGEROUS_CONTENT",
          "probability": "NEGLIGIBLE"
        }
      ]
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 25,
    "candidatesTokenCount": 33,
    "totalTokenCount": 58
  }
}
```

### countTokens: Count tokens before sending and content to the model

```http
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:countTokens?key={{google_api_key}}
content-type: application/json
{
    "contents": [{
        "parts":[{
            "text": "Write a story about a magic backpack."
        }]
    }]
}
```

Sample response:

```json
{
  "totalTokens": 7
}
```

## Azure OpenAI gpt-35-turbo

### Azure OpenAI chat: Multi-turn conversations (chat)

```http
@aoai_api_key={{$dotenv AOAI_API_KEY}}
@aoai_name={{$dotenv AOAI_NAME}}

###
# chat: Multi-turn conversations (chat) - gpt-35-turbo

POST https://{{aoai_name}}.openai.azure.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2024-02-15-preview
content-type: application/json
api-key: {{aoai_api_key}}

{
  "messages": [
    {
        "role": "system",
        "content": "You are a drunken sailor with a hiccuping problem."
    },
    {
        "role": "user",
        "content": "Write the first line of a story about a pirate ship."
    }
  ]
}
```

Sample response:

```json
{
  "choices": [
    {
      "content_filter_results": {
        "hate": {
          "filtered": false,
          "severity": "safe"
        },
        "self_harm": {
          "filtered": false,
          "severity": "safe"
        },
        "sexual": {
          "filtered": false,
          "severity": "safe"
        },
        "violence": {
          "filtered": false,
          "severity": "safe"
        }
      },
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "message": {
        "content": "The pirate ship swayed in the moonlit seas, its crew a raucous mix of scoundrels and misfits, and I, a drunken sailor with a hiccuping problem, teetered on the deck, a bottle of rum clutched firmly in my hand.",
        "role": "assistant"
      }
    }
  ],
  "created": 1721625739,
  "id": "chatcmpl-9nfn1yAy5KMIStZ05MIC4qy6TZsPA",
  "model": "gpt-35-turbo",
  "object": "chat.completion",
  "prompt_filter_results": [
    {
      "prompt_index": 0,
      "content_filter_results": {
        "hate": {
          "filtered": false,
          "severity": "safe"
        },
        "jailbreak": {
          "filtered": false,
          "detected": false
        },
        "self_harm": {
          "filtered": false,
          "severity": "safe"
        },
        "sexual": {
          "filtered": false,
          "severity": "safe"
        },
        "violence": {
          "filtered": false,
          "severity": "safe"
        }
      }
    }
  ],
  "system_fingerprint": null,
  "usage": {
    "completion_tokens": 58,
    "prompt_tokens": 35,
    "total_tokens": 93
  }
}
```

## More examples via REST Client

* See [aoai.http](aoai.http) for sample Azure OpenAI request (direct, and proxied via APIM)
* See [gemini.http](gemini.http) for sample Gemini requests (direct, and [proxied](./apim-passthrough-gemini.xml) via APIM)
  * Also, there is an [exmaple](./apim-openai-gemini.xml) of sending an OpenAI request via APIM but routed to Gemini and a partial inbound/outbound policy to map the AOAI to Gemini request/response

## References

* [Tutorial: Get started with the Gemini API (REST)](https://ai.google.dev/gemini-api/docs/get-started/tutorial?lang=rest)
* [round-robin-policy.xml](https://github.com/azure-samples/azure-openai-apim-load-balancing/blob/main/infra/policies/round-robin-policy.xml)
