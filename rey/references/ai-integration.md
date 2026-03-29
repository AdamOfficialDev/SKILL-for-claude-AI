# REY — AI Integration Guide

> Integrating AI bukan tentang menambahkan ChatGPT. Ini tentang membuat produk yang benar-benar lebih cerdas.

---

## 🤖 AI INTEGRATION PHILOSOPHY

```
WRONG approach:
  "Tambahkan AI" → wrapper around ChatGPT → chatbot di pojok kanan bawah

RIGHT approach:
  Identifikasi friction di product → AI reduces that specific friction
  Examples:
  - User harus manual categorize expenses? → Auto-categorize dengan AI
  - User harus tulis email dari scratch? → AI draft dari bullet points
  - User harus search manually? → Semantic search dengan embeddings
  - User harus analyze data? → Natural language to SQL/chart
```

**REY AI Decision Framework:**
```
Before adding AI, answer:
1. What specific user pain does this solve?
2. What's the accuracy requirement? (95%? 99%? Can wrong answers be corrected?)
3. What's the latency requirement? (Real-time? < 2s? Background processing?)
4. What's the cost per request? (Acceptable?)
5. What happens when AI is wrong? (Fallback strategy?)
```

---

## 🛠️ AI STACK

### Core: Vercel AI SDK
```bash
pnpm add ai @ai-sdk/anthropic @ai-sdk/openai @ai-sdk/google
```

**Why Vercel AI SDK:**
- Unified API across all providers (swap Claude ↔ GPT-4 ↔ Gemini with 1 line change)
- Built-in streaming support (React hooks + RSC)
- Type-safe throughout
- Edge-compatible

### LLM Provider Selection
```typescript
// lib/ai/providers.ts
import { anthropic } from "@ai-sdk/anthropic"
import { openai } from "@ai-sdk/openai"
import { google } from "@ai-sdk/google"

// REY's opinionated defaults per use case:
export const models = {
  // Smart, nuanced tasks: writing, analysis, complex reasoning
  smart: anthropic("claude-opus-4-5"),

  // Everyday tasks: classification, extraction, summarization
  fast: anthropic("claude-haiku-4-5"),

  // Coding tasks
  coding: anthropic("claude-sonnet-4-5"),

  // Vision tasks: analyzing images, screenshots, documents
  vision: openai("gpt-4o"),

  // Embeddings for semantic search / RAG
  embed: openai("text-embedding-3-small"),

  // Large context (processing huge documents)
  largeContext: google("gemini-1.5-pro"),
}
```

---

## 💬 PATTERN 1: AI CHAT (Streaming)

### API Route
```typescript
// app/api/chat/route.ts
import { streamText, Message } from "ai"
import { anthropic } from "@ai-sdk/anthropic"
import { auth } from "@clerk/nextjs/server"
import { rateLimit } from "@/lib/rate-limit"

export const runtime = "edge"
export const maxDuration = 30

export async function POST(req: Request) {
  // Auth check
  const { userId } = await auth()
  if (!userId) return new Response("Unauthorized", { status: 401 })

  // Rate limiting
  const { success } = await rateLimit.limit(userId)
  if (!success) return new Response("Too many requests", { status: 429 })

  const { messages, context }: { messages: Message[]; context?: string } = await req.json()

  const result = streamText({
    model: anthropic("claude-haiku-4-5"),
    system: `You are a helpful assistant for ${context ?? "our product"}.
    
Guidelines:
- Be concise and direct
- If you don't know, say so
- Always suggest next steps
- Format responses with markdown when helpful`,
    messages,
    maxTokens: 1024,
    temperature: 0.7,
    onFinish: async ({ usage, text }) => {
      // Track usage for billing/analytics
      await trackAIUsage({ userId, usage, type: "chat" })
    },
  })

  return result.toDataStreamResponse()
}
```

### React Component
```tsx
// components/features/ai-chat/chat.tsx
"use client"

import { useChat } from "ai/react"
import { useState, useRef, useEffect } from "react"
import { Send, Bot, User, Loader2, RotateCcw } from "lucide-react"
import { Button } from "@/components/ui/button"
import { Textarea } from "@/components/ui/textarea"
import { cn } from "@/lib/utils"
import ReactMarkdown from "react-markdown"
import { motion, AnimatePresence } from "motion/react"

export function AIChat({ initialContext }: { initialContext?: string }) {
  const { messages, input, handleInputChange, handleSubmit, isLoading, error, reload, stop } =
    useChat({
      api: "/api/chat",
      body: { context: initialContext },
      onError: (err) => console.error("Chat error:", err),
    })

  const messagesEndRef = useRef<HTMLDivElement>(null)
  const textareaRef = useRef<HTMLTextAreaElement>(null)

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" })
  }, [messages])

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault()
      handleSubmit(e as any)
    }
  }

  return (
    <div className="flex flex-col h-full">
      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.length === 0 && (
          <div className="flex flex-col items-center justify-center h-full text-muted-foreground">
            <Bot className="h-12 w-12 mb-4 opacity-40" />
            <p className="text-sm">Ask me anything...</p>
          </div>
        )}

        <AnimatePresence initial={false}>
          {messages.map((message) => (
            <motion.div
              key={message.id}
              initial={{ opacity: 0, y: 10 }}
              animate={{ opacity: 1, y: 0 }}
              className={cn(
                "flex gap-3",
                message.role === "user" ? "justify-end" : "justify-start"
              )}
            >
              {message.role === "assistant" && (
                <div className="h-8 w-8 rounded-full bg-primary/10 flex items-center justify-center shrink-0">
                  <Bot className="h-4 w-4 text-primary" />
                </div>
              )}

              <div
                className={cn(
                  "max-w-[80%] rounded-2xl px-4 py-3 text-sm",
                  message.role === "user"
                    ? "bg-primary text-primary-foreground rounded-tr-sm"
                    : "bg-muted rounded-tl-sm prose prose-sm dark:prose-invert"
                )}
              >
                {message.role === "assistant" ? (
                  <ReactMarkdown>{message.content}</ReactMarkdown>
                ) : (
                  message.content
                )}
              </div>

              {message.role === "user" && (
                <div className="h-8 w-8 rounded-full bg-primary flex items-center justify-center shrink-0">
                  <User className="h-4 w-4 text-primary-foreground" />
                </div>
              )}
            </motion.div>
          ))}
        </AnimatePresence>

        {isLoading && (
          <div className="flex gap-3 justify-start">
            <div className="h-8 w-8 rounded-full bg-primary/10 flex items-center justify-center">
              <Bot className="h-4 w-4 text-primary" />
            </div>
            <div className="bg-muted rounded-2xl rounded-tl-sm px-4 py-3">
              <div className="flex gap-1">
                {[0, 1, 2].map((i) => (
                  <motion.div
                    key={i}
                    className="h-2 w-2 rounded-full bg-muted-foreground/50"
                    animate={{ opacity: [0.3, 1, 0.3] }}
                    transition={{ duration: 1.2, repeat: Infinity, delay: i * 0.2 }}
                  />
                ))}
              </div>
            </div>
          </div>
        )}

        {error && (
          <div className="flex items-center gap-2 text-destructive text-sm p-3 rounded-lg bg-destructive/10">
            <span>Something went wrong.</span>
            <Button variant="ghost" size="sm" onClick={() => reload()}>
              <RotateCcw className="h-3 w-3 mr-1" /> Retry
            </Button>
          </div>
        )}

        <div ref={messagesEndRef} />
      </div>

      {/* Input */}
      <form onSubmit={handleSubmit} className="p-4 border-t">
        <div className="flex gap-2 items-end">
          <Textarea
            ref={textareaRef}
            value={input}
            onChange={handleInputChange}
            onKeyDown={handleKeyDown}
            placeholder="Type a message... (Enter to send, Shift+Enter for new line)"
            className="min-h-[44px] max-h-32 resize-none"
            rows={1}
          />
          <Button
            type={isLoading ? "button" : "submit"}
            onClick={isLoading ? stop : undefined}
            size="icon"
            className="shrink-0"
          >
            {isLoading ? (
              <Loader2 className="h-4 w-4 animate-spin" />
            ) : (
              <Send className="h-4 w-4" />
            )}
          </Button>
        </div>
      </form>
    </div>
  )
}
```

---

## 📄 PATTERN 2: STRUCTURED EXTRACTION

### Extract structured data from unstructured text
```typescript
// lib/ai/extract.ts
import { generateObject } from "ai"
import { anthropic } from "@ai-sdk/anthropic"
import { z } from "zod"

// Extract invoice data from PDF text
const invoiceSchema = z.object({
  vendor: z.string().describe("Company or person who issued the invoice"),
  invoiceNumber: z.string().describe("Invoice or bill number"),
  date: z.string().describe("Invoice date in YYYY-MM-DD format"),
  dueDate: z.string().optional().describe("Payment due date in YYYY-MM-DD format"),
  totalAmount: z.number().describe("Total amount due in the invoice currency"),
  currency: z.string().default("USD").describe("Currency code (USD, EUR, etc.)"),
  lineItems: z.array(z.object({
    description: z.string(),
    quantity: z.number().optional(),
    unitPrice: z.number().optional(),
    total: z.number(),
  })).describe("Individual line items"),
  notes: z.string().optional(),
})

export type InvoiceData = z.infer<typeof invoiceSchema>

export async function extractInvoiceData(rawText: string): Promise<InvoiceData> {
  const { object } = await generateObject({
    model: anthropic("claude-haiku-4-5"),
    schema: invoiceSchema,
    prompt: `Extract structured invoice data from the following text:

${rawText}

Extract all available information. For missing fields, use reasonable defaults or omit.`,
  })

  return object
}

// Extract action items from meeting transcript
const meetingNotesSchema = z.object({
  summary: z.string().describe("2-3 sentence executive summary"),
  decisions: z.array(z.string()).describe("Decisions made during meeting"),
  actionItems: z.array(z.object({
    task: z.string(),
    assignee: z.string().optional(),
    dueDate: z.string().optional(),
    priority: z.enum(["high", "medium", "low"]).default("medium"),
  })),
  nextMeetingDate: z.string().optional(),
})

export async function extractMeetingNotes(transcript: string) {
  const { object } = await generateObject({
    model: anthropic("claude-sonnet-4-5"),
    schema: meetingNotesSchema,
    prompt: `Analyze this meeting transcript and extract key information:

${transcript}`,
  })

  return object
}
```

---

## 🔍 PATTERN 3: RAG (Retrieval Augmented Generation)

### Complete RAG Pipeline
```typescript
// lib/ai/rag/pipeline.ts
import { openai } from "@ai-sdk/openai"
import { embed, embedMany, generateText, cosineSimilarity } from "ai"
import { db } from "@/lib/db"
import { documents, embeddings } from "@/lib/db/schema"
import { sql, desc, cosineDistance, gt } from "drizzle-orm"

// Step 1: Chunk text intelligently
export function chunkText(text: string, maxChunkSize = 1000, overlap = 200): string[] {
  const sentences = text.split(/(?<=[.!?])\s+/)
  const chunks: string[] = []
  let currentChunk = ""

  for (const sentence of sentences) {
    if (currentChunk.length + sentence.length > maxChunkSize && currentChunk) {
      chunks.push(currentChunk.trim())
      // Keep overlap from previous chunk
      const words = currentChunk.split(" ")
      const overlapWords = words.slice(-Math.floor(overlap / 6))
      currentChunk = overlapWords.join(" ") + " " + sentence
    } else {
      currentChunk += " " + sentence
    }
  }

  if (currentChunk.trim()) chunks.push(currentChunk.trim())
  return chunks
}

// Step 2: Generate embeddings and store
export async function ingestDocument(content: string, metadata: {
  title: string
  sourceUrl?: string
  userId: string
}) {
  const chunks = chunkText(content)

  // Generate embeddings for all chunks in batch
  const { embeddings: embeddingVectors } = await embedMany({
    model: openai.embedding("text-embedding-3-small"),
    values: chunks,
  })

  // Store in database (using pgvector)
  const [doc] = await db.insert(documents).values({
    title: metadata.title,
    content,
    sourceUrl: metadata.sourceUrl,
    userId: metadata.userId,
  }).returning()

  await db.insert(embeddings).values(
    chunks.map((chunk, i) => ({
      documentId: doc.id,
      chunk,
      embedding: embeddingVectors[i],
      chunkIndex: i,
    }))
  )

  return doc
}

// Step 3: Retrieve relevant chunks for a query
export async function retrieveRelevantChunks(query: string, options: {
  limit?: number
  threshold?: number
  userId?: string
} = {}) {
  const { limit = 5, threshold = 0.7, userId } = options

  // Embed the query
  const { embedding: queryEmbedding } = await embed({
    model: openai.embedding("text-embedding-3-small"),
    value: query,
  })

  // Cosine similarity search with pgvector
  const results = await db.execute(sql`
    SELECT
      e.chunk,
      e.chunk_index,
      d.title,
      d.source_url,
      1 - (e.embedding <=> ${JSON.stringify(queryEmbedding)}::vector) as similarity
    FROM embeddings e
    JOIN documents d ON e.document_id = d.id
    WHERE 1 - (e.embedding <=> ${JSON.stringify(queryEmbedding)}::vector) > ${threshold}
    ${userId ? sql`AND d.user_id = ${userId}` : sql``}
    ORDER BY similarity DESC
    LIMIT ${limit}
  `)

  return results.rows as Array<{
    chunk: string
    chunk_index: number
    title: string
    source_url: string | null
    similarity: number
  }>
}

// Step 4: Generate response with retrieved context
export async function ragQuery(query: string, userId: string): Promise<{
  answer: string
  sources: Array<{ title: string; url: string | null; similarity: number }>
}> {
  const relevantChunks = await retrieveRelevantChunks(query, { userId })

  if (relevantChunks.length === 0) {
    return {
      answer: "I couldn't find relevant information in your documents to answer this question.",
      sources: [],
    }
  }

  const context = relevantChunks
    .map((chunk, i) => `[Source ${i + 1}: ${chunk.title}]\n${chunk.chunk}`)
    .join("\n\n---\n\n")

  const { text } = await generateText({
    model: anthropic("claude-sonnet-4-5"),
    system: `You are a helpful assistant that answers questions based on provided documents.
    
Rules:
- Only use information from the provided context
- If the answer is not in the context, say so clearly
- Cite your sources using [Source N] notation
- Be concise and accurate`,
    prompt: `Context from documents:
${context}

Question: ${query}

Provide a helpful answer based on the context above.`,
    maxTokens: 1024,
  })

  const sources = relevantChunks.map(chunk => ({
    title: chunk.title,
    url: chunk.source_url,
    similarity: chunk.similarity,
  }))

  return { answer: text, sources }
}
```

### pgvector Schema
```typescript
// lib/db/schema/embeddings.ts
import { pgTable, uuid, text, integer, index, timestamp } from "drizzle-orm/pg-core"
import { vector } from "drizzle-orm/pg-core"
import { sql } from "drizzle-orm"

export const documents = pgTable("documents", {
  id: uuid("id").primaryKey().defaultRandom(),
  userId: text("user_id").notNull(),
  title: text("title").notNull(),
  content: text("content").notNull(),
  sourceUrl: text("source_url"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
})

export const embeddings = pgTable("embeddings", {
  id: uuid("id").primaryKey().defaultRandom(),
  documentId: uuid("document_id").references(() => documents.id, { onDelete: "cascade" }).notNull(),
  chunk: text("chunk").notNull(),
  chunkIndex: integer("chunk_index").notNull(),
  // 1536 dimensions for text-embedding-3-small
  embedding: vector("embedding", { dimensions: 1536 }).notNull(),
}, (table) => ({
  // IVFFlat index for approximate nearest neighbor search
  embeddingIndex: index("embedding_idx").using(
    "ivfflat",
    table.embedding.op("vector_cosine_ops")
  ).with({ lists: 100 }),
}))
```

```sql
-- Migration: Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;
```

---

## 🤖 PATTERN 4: AI AGENTS

### Simple Agent with Tool Use
```typescript
// lib/ai/agents/research-agent.ts
import { generateText, tool } from "ai"
import { anthropic } from "@ai-sdk/anthropic"
import { z } from "zod"

export async function researchAgent(topic: string): Promise<string> {
  const { text, steps } = await generateText({
    model: anthropic("claude-sonnet-4-5"),
    maxSteps: 5, // Allow up to 5 tool calls
    system: `You are a research assistant. Use available tools to research topics thoroughly.
Always verify information from multiple sources when possible.`,
    prompt: `Research the following topic and provide a comprehensive summary: ${topic}`,

    tools: {
      searchWeb: tool({
        description: "Search the web for current information",
        parameters: z.object({
          query: z.string().describe("Search query"),
        }),
        execute: async ({ query }) => {
          // Implement actual web search here
          // Could use Serper, Tavily, or Brave Search API
          const results = await searchWebAPI(query)
          return results
        },
      }),

      readUrl: tool({
        description: "Read and extract content from a URL",
        parameters: z.object({
          url: z.string().url().describe("URL to read"),
        }),
        execute: async ({ url }) => {
          const response = await fetch(url)
          const html = await response.text()
          return extractTextFromHTML(html).slice(0, 3000) // Limit context
        },
      }),

      saveNote: tool({
        description: "Save an important note or finding",
        parameters: z.object({
          title: z.string(),
          content: z.string(),
        }),
        execute: async ({ title, content }) => {
          // Save to DB or memory
          return { saved: true, id: crypto.randomUUID() }
        },
      }),
    },
  })

  return text
}
```

### Multi-Step Agent with State
```typescript
// lib/ai/agents/task-agent.ts
import { generateText, tool, CoreMessage } from "ai"
import { anthropic } from "@ai-sdk/anthropic"
import { z } from "zod"

type AgentState = {
  task: string
  completedSteps: string[]
  findings: Record<string, unknown>
}

export async function taskAgent(task: string, onProgress?: (step: string) => void) {
  const state: AgentState = {
    task,
    completedSteps: [],
    findings: {},
  }

  const messages: CoreMessage[] = [
    { role: "user", content: task }
  ]

  // Agentic loop
  while (true) {
    const result = await generateText({
      model: anthropic("claude-opus-4-5"),
      messages,
      system: `You are an autonomous task agent. Complete tasks step by step.
Current progress: ${state.completedSteps.join(" → ")}
Always verify your work before declaring completion.`,
      tools: {
        executeStep: tool({
          description: "Execute a specific step towards completing the task",
          parameters: z.object({
            step: z.string(),
            action: z.string(),
          }),
          execute: async ({ step, action }) => {
            onProgress?.(step)
            state.completedSteps.push(step)
            // Execute actual action...
            return { success: true, result: `Executed: ${action}` }
          },
        }),
        taskComplete: tool({
          description: "Mark the task as complete",
          parameters: z.object({
            summary: z.string(),
          }),
          execute: async ({ summary }) => {
            return { done: true, summary }
          },
        }),
      },
    })

    // Push assistant response to messages
    messages.push({ role: "assistant", content: result.content })

    // Check if task is complete
    const doneCall = result.toolCalls.find(call => call.toolName === "taskComplete")
    if (doneCall || result.finishReason === "stop") {
      return {
        success: true,
        steps: state.completedSteps,
        result: result.text,
      }
    }

    // Push tool results back
    messages.push({
      role: "tool",
      content: result.toolResults,
    })
  }
}
```

---

## 📊 PATTERN 5: AI-POWERED FEATURES

### Smart Autocomplete
```typescript
// lib/ai/autocomplete.ts
import { streamText } from "ai"
import { anthropic } from "@ai-sdk/anthropic"

export async function generateAutocomplete(
  prefix: string,
  context: string,
  maxTokens = 50
): Promise<string> {
  const { text } = await generateText({
    model: anthropic("claude-haiku-4-5"),
    prompt: `Complete this text naturally. Only output the completion, not the original text.

Context: ${context}
Text to complete: "${prefix}"

Completion (continue from exactly where the text ends):`,
    maxTokens,
    temperature: 0.7,
    stopSequences: ["\n", ".", "!", "?"],
  })

  return text
}
```

### Sentiment Analysis
```typescript
// lib/ai/classify.ts
import { generateObject } from "ai"
import { anthropic } from "@ai-sdk/anthropic"
import { z } from "zod"

const sentimentSchema = z.object({
  sentiment: z.enum(["positive", "negative", "neutral", "mixed"]),
  score: z.number().min(-1).max(1).describe("Sentiment score from -1 (very negative) to 1 (very positive)"),
  emotions: z.array(z.string()).describe("Detected emotions"),
  keyPhrases: z.array(z.string()).describe("Key phrases that influenced the sentiment"),
})

export async function analyzeSentiment(text: string) {
  const { object } = await generateObject({
    model: anthropic("claude-haiku-4-5"),
    schema: sentimentSchema,
    prompt: `Analyze the sentiment of the following text:\n\n${text}`,
  })
  return object
}

// Content moderation
const moderationSchema = z.object({
  safe: z.boolean(),
  categories: z.array(z.string()).describe("Problematic categories if not safe"),
  confidence: z.number().min(0).max(1),
})

export async function moderateContent(text: string) {
  const { object } = await generateObject({
    model: anthropic("claude-haiku-4-5"),
    schema: moderationSchema,
    prompt: `Check if this content is safe for a professional platform:\n\n${text}`,
  })
  return object
}
```

### AI-Powered Search (Semantic)
```typescript
// lib/ai/search.ts
import { embed } from "ai"
import { openai } from "@ai-sdk/openai"
import { db } from "@/lib/db"
import { sql } from "drizzle-orm"

export async function semanticSearch(query: string, options: {
  table: string
  contentColumn: string
  embeddingColumn: string
  limit?: number
  filter?: Record<string, unknown>
}) {
  const { limit = 10 } = options

  // Embed the search query
  const { embedding } = await embed({
    model: openai.embedding("text-embedding-3-small"),
    value: query,
  })

  // Vector similarity search
  const results = await db.execute(sql`
    SELECT *,
      1 - (${sql.identifier(options.embeddingColumn)} <=> ${JSON.stringify(embedding)}::vector) as similarity
    FROM ${sql.identifier(options.table)}
    ORDER BY similarity DESC
    LIMIT ${limit}
  `)

  return results.rows
}
```

---

## 💰 AI COST OPTIMIZATION

```typescript
// lib/ai/cost-tracker.ts

// Token cost estimates (update regularly)
const COSTS = {
  "claude-haiku-4-5": { input: 0.00025, output: 0.00125 },      // per 1K tokens
  "claude-sonnet-4-5": { input: 0.003, output: 0.015 },
  "claude-opus-4-5": { input: 0.015, output: 0.075 },
  "gpt-4o": { input: 0.005, output: 0.015 },
  "text-embedding-3-small": { input: 0.00002, output: 0 },
}

export function estimateCost(model: string, inputTokens: number, outputTokens: number): number {
  const prices = COSTS[model as keyof typeof COSTS]
  if (!prices) return 0
  return (inputTokens / 1000) * prices.input + (outputTokens / 1000) * prices.output
}

// Caching to avoid redundant AI calls
import { createHash } from "crypto"

const aiCache = new Map<string, { result: unknown; timestamp: number }>()
const CACHE_TTL = 1000 * 60 * 60 // 1 hour

export function withAICache<T>(
  key: string,
  fn: () => Promise<T>,
  ttl = CACHE_TTL
): Promise<T> {
  const cacheKey = createHash("sha256").update(key).digest("hex")
  const cached = aiCache.get(cacheKey)

  if (cached && Date.now() - cached.timestamp < ttl) {
    return Promise.resolve(cached.result as T)
  }

  return fn().then(result => {
    aiCache.set(cacheKey, { result, timestamp: Date.now() })
    return result
  })
}

// Usage
const summary = await withAICache(
  `summarize:${documentId}`,
  () => summarizeDocument(content)
)
```

---

## 🛡️ AI SAFETY PATTERNS

```typescript
// lib/ai/safety.ts

// Prompt injection detection
export async function detectPromptInjection(input: string): Promise<boolean> {
  const suspiciousPatterns = [
    /ignore previous instructions/i,
    /system prompt/i,
    /you are now/i,
    /disregard your/i,
    /forget everything/i,
    /jailbreak/i,
  ]

  return suspiciousPatterns.some(pattern => pattern.test(input))
}

// Input sanitization for AI prompts
export function sanitizeAIInput(input: string): string {
  return input
    .slice(0, 4000) // Limit input length
    .replace(/<[^>]*>/g, "") // Remove HTML
    .trim()
}

// Output validation
export function validateAIOutput<T extends z.ZodSchema>(
  output: string,
  schema: T
): z.infer<T> | null {
  try {
    return schema.parse(JSON.parse(output))
  } catch {
    return null
  }
}

// Wrap AI calls with safety checks
export async function safeAICall<T>(
  userInput: string,
  fn: (sanitizedInput: string) => Promise<T>
): Promise<T | null> {
  if (await detectPromptInjection(userInput)) {
    console.warn("Potential prompt injection detected")
    return null
  }

  const sanitized = sanitizeAIInput(userInput)
  return fn(sanitized)
}
```
