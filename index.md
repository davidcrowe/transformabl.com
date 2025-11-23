---
layout: default
title: transformabl
---

# transformabl

**content transformation and safety preprocessing for llm and agentic apps**

before a request can be validated or routed, its content must be transformed into a safe, structured, model-ready form.

transformabl defines this layer.

## at a glance

transformabl is the **content transformation and safety preprocessing gateway** for llm apps.  

it lets you:

- detect and redact pii  
- classify content (sensitive, legal, unsafe)  
- segment inputs into safe vs unsafe regions  
- normalize, clean, and structure requests  
- generate metadata for routing and policy decisions  

> 📦 **implementation**: [`ai-content-gateway`](https://github.com/davidcrowe/gatewaystack) (roadmap)

transformabl is identity-agnostic; policy decisions happen in validatabl.

## why now?

as llm apps operate on personal, financial, or regulated data, preprocessing becomes critical.  

policies depend on what the content *is*, not just who sent it.

transformabl prepares content for the rest of the gatewaystack pipeline.

## designing the content transformation layer

### within the shared requestcontext

all gatewaystack modules operate on a shared [`RequestContext`](https://github.com/davidcrowe/gatewaystack/blob/main/docs/reference/interfaces.md) object.

**transformabl is responsible for**:

- **reading**: `identity` (optional), raw `content`, upstream hints from client  
- **writing**: `metadata` (classification, pii flags, risk score, topics), transformed `content` (redacted/pseudonymized), transformation logs

transformabl receives raw user input and applies transformations that:

- protect user privacy  
- enable safety policies  
- improve routing and model selection  
- reduce downstream risk  

it ensures that everything passed to validatabl and proxyabl is safe, structured, and explainable.

## bidirectional transformation

transformabl operates on both requests and responses:

**request transformation:**
- redact pii before sending to llm  
- detect prompt injection  
- normalize structure  

**response transformation:**
- remove hallucinated pii  
- filter unsafe generated content  
- validate response structure  

## pseudonymization and reversibility

transformabl supports both:

- **one-way redaction** — pii is permanently removed  
- **reversible pseudonymization** — pii is replaced with tokens and restored in responses  

this enables workflows where the llm processes anonymized data, but the end user sees the original context.

## classification taxonomy

transformabl detects:

**safety risks:**
- prompt injection / jailbreaks  
- malicious code  
- hate speech / abuse  

**regulated content:**
- hipaa (healthcare data)  
- pci (payment data)  
- gdpr (eu personal data)  
- coppa (children's data)  

**business policies:**
- competitor mentions  
- proprietary information  
- confidential data  

## the core functions

**1. `detectPII` — identify personal or sensitive data**  
emails, phone numbers, addresses, ids, biometrics.

**2. `redactPII` — remove or mask detected pii**  
full or partial redaction options.

**3. `classifyContent` — detect unsafe / legal / regulated categories**  

**4. `segmentInput` — split content into safe, unsafe, and neutral regions**  

**5. `normalize` — formatting cleanup, whitespace, casing, structure**  

**6. `extractMetadata` — topics, intent, sentiment, risk**  

**7. `analyzeForRouting` — suggest optimal provider/model/tool based on content**  
(e.g., contains phi → route to hipaa-compliant model; python code → route to code-optimized model)

**8. `pseudonymize` — reversible tokenization of pii regions**  

**9. `rehydrate` — restore pseudonymized values on response**  

## what transformabl does

- preprocesses content for safety  
- prepares metadata for governance decisions  
- anonymizes and pseudonymizes sensitive regions  
- emits structured transformation logs for explicabl  
- populates the `metadata` field in `RequestContext`

## transformabl works with

- `identifiabl` to authenticate users
- `validatabl` to enforce permissions
- `limitabl` to apply rate limits or quotas
- `proxyabl` to perform provider routing
- `explicabl` to store or ship audit logs

## performance considerations

running pii detection, content classification, and metadata extraction on every request can be expensive:

- **latency overhead**: 50–200ms per request depending on models/rules  
- **cost overhead**: if using secondary llms or external services  
- **infrastructure load**: additional compute for classification models  

transformabl pipelines are **configurable** — enable only what you actually need.

**common patterns:**

- enable pii detection and redaction for regulated workloads  
- run full classification only on high-risk tenants or environments  
- cache transformation results for repeated inputs where possible  

**optimization strategies:**
```yaml
transformations:
  pii_detection:
    enabled: true
    cache_ttl: 300s  # cache results for repeated content
  
  content_classification:
    enabled_for:
      - org_healthcare
      - org_finance
    skip_for:
      - tier_free  # skip classification for free tier
  
  routing_analysis:
    enabled: true
    use_lightweight_model: true  # faster, less accurate
```

## end to end flow
```text
user
   → identifiabl       (who is calling?)
   → transformabl      (prepare, clean, classify, anonymize)
   → validatabl        (is this allowed?)
   → limitabl          (can they afford it? pre-flight constraints)
   → proxyabl          (where does it go? execute)
   → llm provider      (model call)
   → [limitabl]        (deduct actual usage, update quotas/budgets)
   → explicabl         (what happened?)
   → response
```

every request becomes safer and more structured before governance rules run.

## integrates with your existing stack

transformabl plugs into gatewaystack and your existing llm stack without requiring application-level changes. it exposes http middleware and sdk hooks for:

- chatgpt apps sdk  
- model context protocol (mcp)  
- oauth2 / oidc identity providers  
- any llm provider (openai, anthropic, google, internal models)  

## getting started

**for transformation configuration**:  
→ [PII detection patterns](https://github.com/davidcrowe/gatewaystack/blob/main/docs/examples/pii-detection.md)  
→ [content classification guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/content-classification.md)  
→ [pseudonymization strategies](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/pseudonymization.md)

**for compliance use cases**:  
→ [HIPAA compliance setup](https://github.com/davidcrowe/gatewaystack/blob/main/docs/examples/hipaa-compliance.md)  
→ [GDPR data handling](https://github.com/davidcrowe/gatewaystack/blob/main/docs/examples/gdpr-compliance.md)

**for implementation**:  
→ [integration guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/integration.md)

## links

want to explore the full gatewaystack architecture?  
→ [view the gatewaystack github repo](https://github.com/davidcrowe/gatewaystack)

want to contact us for enterprise deployments?  
→ [reducibl applied ai studio](https://reducibl.com)

<div class="arch-diagram">
  <div class="arch-row">
    <div class="arch-node">
      <div class="arch-node-title">app / agent</div>
      <div class="arch-node-sub">chat ui · internal tool · agent runtime</div>
    </div>

    <div class="arch-arrow">→</div>

    <div class="arch-node arch-node-gateway">
      <div class="arch-node-title">gatewaystack</div>
      <div class="arch-node-sub">user-scoped trust &amp; governance gateway</div>

      <div class="arch-pill-row">
        <span class="arch-pill">identifiabl</span>
        <span class="arch-pill">transformabl</span>
        <span class="arch-pill">validatabl</span>
        <span class="arch-pill">limitabl</span>
        <span class="arch-pill">proxyabl</span>
        <span class="arch-pill">explicabl</span>
      </div>
    </div>

    <div class="arch-arrow">→</div>

    <div class="arch-node">
      <div class="arch-node-title">llm providers</div>
      <div class="arch-node-sub">openai · anthropic · internal models</div>
    </div>
  </div>

  <p class="arch-caption">
    every request flows from your app through gatewaystack's modules before it reaches an llm provider —
    <strong>identified, transformed, validated, constrained, routed, and audited.</strong>
  </p>
</div>