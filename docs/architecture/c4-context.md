# C4 — Context Diagram

ClaimPilot sits between the people filing/handling claims and the external systems it depends on.

```mermaid
C4Context
    title ClaimPilot — System Context

    Person(customer, "Customer", "Files a claim, uploads supporting documents, receives a decision letter")
    Person(adjuster, "Adjuster", "Reviews agent recommendations and approves or rejects claims")

    System(claimpilot, "ClaimPilot", "AI-powered insurance claims platform. Intake, policy coverage check, fraud scoring, agent recommendation, human approval, decision letter.")

    System_Ext(aifoundry, "Azure AI Foundry", "Hosts the chat and embedding models used for RAG, agent reasoning and summaries")
    System_Ext(email, "Email / Notification Provider", "Delivers claim status updates and decision letters to customers")
    System_Ext(blob, "Azure Blob Storage", "Stores uploaded claim documents and images")
    System_Ext(entra, "Microsoft Entra ID", "Authenticates users and issues managed/workload identities — no secrets in code")

    Rel(customer, claimpilot, "Files claims, uploads documents, views status", "HTTPS")
    Rel(adjuster, claimpilot, "Reviews queue, approves/rejects claims", "HTTPS")
    Rel(claimpilot, aifoundry, "Chat completions, embeddings", "HTTPS / SDK")
    Rel(claimpilot, email, "Sends notifications and decision letters", "SMTP/API")
    Rel(claimpilot, blob, "Stores/retrieves documents", "HTTPS")
    Rel(claimpilot, entra, "AuthN/AuthZ, Managed Identity, Workload Identity", "OAuth2/OIDC")
```

## One-sentence summary per box

- **Customer** — files a claim and supplies documents; the only external actor who doesn't log in as staff.
- **Adjuster** — the human in the loop; makes the final approve/reject call on every agent recommendation.
- **ClaimPilot** — the system being built: nine services behind a gateway, described further in the Container diagram.
- **Azure AI Foundry** — the only source of LLM intelligence; nothing in ClaimPilot calls a model directly without going through it.
- **Email/Notification Provider** — a dumb delivery channel; ClaimPilot never stores notification content there.
- **Azure Blob Storage** — the only place raw claim documents live; the Document Service never persists file bytes in PostgreSQL.
- **Microsoft Entra ID** — every identity in the system (user, pod, service) is issued and verified here; this is why there are no secrets in code.
