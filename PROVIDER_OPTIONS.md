# Provider-Specific Options

This document explains the provider-specific options system that allows you to pass provider-specific configuration to AI adapters in a type-safe manner.

## Overview

Each AI provider (OpenAI, Anthropic, Gemini, etc.) has unique features and configuration options. The `providerOptions` field allows you to access these provider-specific features while maintaining full TypeScript type safety.

## Key Features

✅ **Type-Safe**: TypeScript provides autocomplete and type checking based on the selected adapter  
✅ **Provider-Specific**: Each adapter defines its own option types  
✅ **No Global Types**: Uses TypeScript's type inference without module augmentation  
✅ **Flexible**: Works with chat, text generation, and image generation  
✅ **Well-Documented**: Based on official AI SDK documentation

## How It Works

### 1. Adapter Type Parameters

Each adapter class extends `BaseAdapter` with three type parameters:

```typescript
export class OpenAIAdapter extends BaseAdapter<
  typeof OPENAI_MODELS,           // Available text models
  typeof OPENAI_IMAGE_MODELS,     // Available image models
  OpenAIProviderOptions           // Provider-specific options type
> {
  name = "openai" as const;
  // ...
}
```

### 2. Type Inference

The AI class automatically extracts the provider options type from each adapter and makes it available based on the adapter's internal `name`:

```typescript
// The AI class extracts:
// - Adapter's internal name: "openai"
// - Provider options type: OpenAIProviderOptions

// Then creates a type like:
type ProviderOptions = {
  openai?: OpenAIProviderOptions;
}
```

### 3. Usage

When you call an AI method with a specific adapter, TypeScript automatically knows which provider options are available:

```typescript
await ai.chat({
  adapter: "openai",  // TypeScript knows this is OpenAI
  model: "gpt-4o",
  messages: [{ role: "user", content: "Hello" }],
  providerOptions: {
    openai: {           // ✅ Autocomplete shows OpenAI-specific options
      reasoningSummary: "detailed",
      textVerbosity: "high",
      // ... other OpenAI options
    },
  },
});
```

## OpenAI Provider Options

Based on: https://ai-sdk.dev/providers/ai-sdk-providers/openai

### Chat & Text Generation Options

```typescript
interface OpenAIProviderOptions {
  // Parallel tool calling
  parallelToolCalls?: boolean; // Default: true
  
  // Storage for distillation
  store?: boolean; // Default: true
  metadata?: Record<string, string>;
  
  // Reasoning models (gpt-4o, gpt-5, etc.)
  reasoningEffort?: 'minimal' | 'low' | 'medium' | 'high';
  reasoningSummary?: 'auto' | 'detailed';
  
  // Response control
  textVerbosity?: 'low' | 'medium' | 'high';
  serviceTier?: 'auto' | 'flex' | 'priority' | 'default';
  
  // Advanced options
  maxToolCalls?: number;
  maxCompletionTokens?: number;
  strictJsonSchema?: boolean;
  include?: string[]; // e.g., ['file_search_call.results']
  
  // Monitoring & caching
  user?: string; // Unique user identifier
  promptCacheKey?: string; // Manual cache control
  safetyIdentifier?: string; // Usage policy violation detection
  
  // Token control
  logitBias?: Record<number, number>;
  logprobs?: boolean | number;
  
  // Advanced features
  prediction?: { type: 'content'; content: string };
  structuredOutputs?: boolean;
  
  // Message-level options
  imageDetail?: 'high' | 'low' | 'auto'; // For images in messages
}
```

### Image Generation Options

```typescript
interface OpenAIImageProviderOptions {
  quality?: 'standard' | 'hd'; // DALL-E 3 only
  style?: 'natural' | 'vivid'; // DALL-E 3 only
  seed?: number; // For reproducibility (DALL-E 3 only)
}
```

### Examples

#### Reasoning Models

```typescript
await ai.chat({
  adapter: "openai",
  model: "gpt-4o",
  messages: [{ role: "user", content: "Explain TypeScript generics" }],
  providerOptions: {
    openai: {
      reasoningSummary: "detailed", // Get detailed reasoning
      reasoningEffort: "high",      // Maximum reasoning effort
      textVerbosity: "high",        // Verbose response
    },
  },
});
```

#### Tool Calling with Options

```typescript
await ai.chat({
  adapter: "openai",
  model: "gpt-4o",
  messages: [{ role: "user", content: "Calculate something" }],
  tools: ["calculator"],
  providerOptions: {
    openai: {
      parallelToolCalls: false, // Execute tools sequentially
      maxToolCalls: 5,          // Limit total tool calls
      user: "user-123",         // For monitoring
    },
  },
});
```

#### Prompt Caching

```typescript
await ai.chat({
  adapter: "openai",
  model: "gpt-4o",
  messages: [
    { role: "system", content: "Long system prompt..." }, // >1024 tokens
    { role: "user", content: "Question" },
  ],
  providerOptions: {
    openai: {
      promptCacheKey: "my-system-prompt-v1", // Manual cache control
      serviceTier: "flex", // 50% cheaper, higher latency
    },
  },
});
```

#### Log Probabilities

```typescript
await ai.chat({
  adapter: "openai",
  model: "gpt-4o",
  messages: [{ role: "user", content: "Is TypeScript good?" }],
  providerOptions: {
    openai: {
      logprobs: 5, // Get top 5 token probabilities
      logitBias: {
        9820: 10,   // Increase "yes" likelihood
        2201: -10,  // Decrease "no" likelihood
      },
    },
  },
});
```

#### Image Generation

```typescript
await ai.image({
  adapter: "openai",
  model: "dall-e-3",
  prompt: "A futuristic guitar",
  providerOptions: {
    openai: {
      quality: "hd",      // High quality
      style: "vivid",     // Vivid colors
      seed: 42,           // Reproducible
    },
  },
});
```

## Anthropic Provider Options

Based on: https://ai-sdk.dev/providers/ai-sdk-providers/anthropic

### Chat Options

```typescript
interface AnthropicProviderOptions {
  // Reasoning support
  thinking?: {
    type: 'enabled';
    budgetTokens: number; // e.g., 12000
  };
  
  // Prompt caching
  cacheControl?: {
    type: 'ephemeral';
    ttl?: '5m' | '1h'; // Default: '5m'
  };
  
  // Request control
  sendReasoning?: boolean; // Default: true
}
```

### Examples

#### Reasoning Models

```typescript
await ai.chat({
  adapter: "anthropic",
  model: "claude-opus-4-20250514",
  messages: [{ role: "user", content: "Complex reasoning task" }],
  providerOptions: {
    anthropic: {
      thinking: {
        type: "enabled",
        budgetTokens: 12000, // Allocate tokens for reasoning
      },
      sendReasoning: true,
    },
  },
});
```

#### Prompt Caching

```typescript
const longDocument = "..."; // >2048 tokens

await ai.chat({
  adapter: "anthropic",
  model: "claude-3-5-sonnet-20241022",
  messages: [
    { role: "user", content: `Analyze: ${longDocument}` },
  ],
  providerOptions: {
    anthropic: {
      cacheControl: {
        type: "ephemeral",
        ttl: "1h", // Cache for 1 hour instead of 5 minutes
      },
    },
  },
});
```

## Important Notes

### Provider Name vs. Adapter Name

⚠️ The `providerOptions` key must match the adapter's internal `name` property, not your custom adapter name:

```typescript
const ai = new AI({
  adapters: {
    myOpenAI: new OpenAIAdapter({ apiKey: "..." }), // Custom name
  },
});

// ✅ Correct - use adapter's internal name
await ai.chat({
  adapter: "myOpenAI",
  model: "gpt-4",
  providerOptions: {
    openai: { ... }, // ✅ Use "openai" (adapter's internal name)
  },
});

// ❌ Incorrect
await ai.chat({
  adapter: "myOpenAI",
  model: "gpt-4",
  providerOptions: {
    myOpenAI: { ... }, // ❌ Won't work
  },
});
```

The adapter's internal name is defined in the adapter class:

```typescript
export class OpenAIAdapter extends BaseAdapter {
  name = "openai"; // ← This is the key to use in providerOptions
  // ...
}
```

### Type Safety

TypeScript will catch invalid options:

```typescript
// ❌ TypeScript error - thinking doesn't exist on OpenAI
await ai.chat({
  adapter: "openai",
  model: "gpt-4",
  providerOptions: {
    openai: {
      thinking: { ... }, // ERROR!
    },
  },
});

// ❌ TypeScript error - reasoningSummary doesn't exist on Anthropic
await ai.chat({
  adapter: "anthropic",
  model: "claude-3-opus",
  providerOptions: {
    anthropic: {
      reasoningSummary: "detailed", // ERROR!
    },
  },
});
```

## Adding Provider Options to Custom Adapters

If you create a custom adapter, you can add provider-specific options by passing them as the third type parameter to `BaseAdapter`:

1. Define your options interface:

```typescript
export interface MyCustomProviderOptions {
  customOption1?: string;
  customOption2?: number;
}
```

2. Pass it to BaseAdapter as the third type parameter:

```typescript
import { BaseAdapter } from "@tanstack/ai";

const MY_CUSTOM_MODELS = ["custom-model-1", "custom-model-2"] as const;

export class MyCustomAdapter extends BaseAdapter<
  typeof MY_CUSTOM_MODELS,      // Models type
  readonly [],                   // Image models type (empty if not supported)
  MyCustomProviderOptions        // Provider options type
> {
  name = "mycustom" as const;    // Internal name (used in providerOptions key)
  models = MY_CUSTOM_MODELS;
  
  // ... implementation
}
```

3. Extract and use options in your adapter methods:

```typescript
export class MyCustomAdapter extends BaseAdapter<
  typeof MY_CUSTOM_MODELS,
  readonly [],
  MyCustomProviderOptions
> {
  name = "mycustom" as const;
  models = MY_CUSTOM_MODELS;

  async chatCompletion(options: ChatCompletionOptions): Promise<ChatCompletionResult> {
    // TypeScript knows this is MyCustomProviderOptions based on the adapter instance
    const providerOpts = options.providerOptions?.mycustom as MyCustomProviderOptions | undefined;

    // Use providerOpts to configure your API calls
    if (providerOpts?.customOption1) {
      // Apply custom option
    }

    // ... rest of implementation
  }
}
```

4. Type safety works automatically:

```typescript
const ai = new AI({
  adapters: {
    custom: new MyCustomAdapter({ /* config */ }),
  },
});

await ai.chat({
  adapter: "custom",
  model: "custom-model-1",
  messages: [...],
  providerOptions: {
    mycustom: {          // ✅ TypeScript infers MyCustomProviderOptions here
      customOption1: "value",
      customOption2: 42,
    },
  },
});
```

No global module augmentation needed! TypeScript extracts the provider options type directly from your adapter instance.

## References

- [OpenAI Provider Documentation](https://ai-sdk.dev/providers/ai-sdk-providers/openai)
- [Anthropic Provider Documentation](https://ai-sdk.dev/providers/ai-sdk-providers/anthropic)
- [AI SDK Core Documentation](https://ai-sdk.dev/docs/ai-sdk-core)

## Summary

Provider-specific options give you access to the full power of each AI provider while maintaining type safety and a clean API. Use them to:

- Control reasoning behavior (OpenAI, Anthropic)
- Optimize costs with caching (OpenAI, Anthropic)
- Fine-tune responses with verbosity controls (OpenAI)
- Manage tool execution (OpenAI)
- Access advanced features unique to each provider

The type system ensures you only use valid options for each provider, preventing runtime errors and providing excellent IDE support.
