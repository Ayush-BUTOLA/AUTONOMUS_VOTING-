Privacy-Preserving Voting Infrastructure — Master Project Brief
1. Project Intent
Build a reusable privacy-preserving voting infrastructure platform that can be integrated by organizations such as:
- Colleges/universities
- DAOs
- Web3 communities
- Other organizations with their own membership/eligibility systems
The platform should allow an organization to create and manage elections while providing:
1. Eligibility verification
2. Privacy-preserving identity verification
3. Anonymous voting
4. Double-vote prevention
5. Verifiable tallying
6. Blockchain-backed auditability
7. API/SDK integration
8. MCP integration for agentic workflows
The platform is NOT supposed to become the authority that decides who belongs to an organization.
The organization remains the source of truth for eligibility.
2. Core Problem
Traditional online voting has several problems:
- Centralized databases can be compromised.
- Voter identity can be linked to votes.
- It is difficult for voters to independently verify results.
- Double voting can occur if identity management is weak.
- Organizations have different ways of managing members.
- DAO voting systems are often tightly coupled to specific governance systems.
The goal is to provide a reusable voting infrastructure where:
An organization can verify that a person is eligible to vote, while the voting system does not need to reveal the person's identity when they cast their vote.

The system should also allow anyone to verify that the final result was produced according to the election rules.
3. Core Privacy Model
The system separates identity, eligibility, and voting.
The intended privacy boundary is:
Organization
    ↓
Knows who its members are

Voting platform
    ↓
Knows that an anonymous credential is eligible

Blockchain
    ↓
Stores cryptographic proofs / commitments / election state

Public
    ↓
Can verify election integrity and final result

Nobody should normally be able to see:

    Person → Candidate they voted for
Do NOT create an admin system where an administrator can routinely decode every anonymous voter and see their vote.
If identity recovery is ever required, it should be a separate exceptional process with strong authorization and auditing.
4. Main Architecture
                 COLLEGE / DAO
                       │
                       ↓
              API / SDK / MCP
                       │
                       ↓
             ELIGIBILITY ADAPTER
                       │
                       ↓
              ELIGIBILITY CHECK
                       │
                       ↓
              DID / CREDENTIAL
                       │
                       ↓
                  ZK PROOF
                       │
                       ↓
             ELECTION NULLIFIER
                       │
                       ↓
              PRIVATE VOTING
                   PROTOCOL
                       │
                       ↓
              MST BLOCKCHAIN
                       │
                       ↓
              SMART CONTRACT
                       │
                       ↓
                    TALLY
                       │
                       ↓
                ZK TALLY PROOF
                       │
                       ↓
                   RESULT
                       │
                       ↓
                PUBLIC AUDIT
5. Layer 1 — Organization Layer
Supported organizations initially:
College
Possible eligibility sources:
- Institutional email
- Student ID
- College-provided verification API/database
Example:
student@college.edu
        ↓
College verifies student
        ↓
Eligible
DAO
Possible eligibility sources:
- Wallet ownership
- Governance token
- Membership NFT
- DAO allowlist
- Governance contract
- DAO API
Example:
Wallet
  ↓
DAO governance contract
  ↓
Membership check
  ↓
Eligible
Important:
The DAO's existing membership system is the source of truth.
The platform should not require DAO members to submit Aadhaar or unnecessary real-world identity information.
6. Layer 2 — Integration Layer
This layer connects the platform to the organization's existing systems.
The platform should support:
- REST API
- SDK
- MCP
The integration layer should use adapters.
Example:
College System ──→ College Adapter ──┐
                                     │
DAO Contract ────→ DAO Adapter ──────┤
                                     ↓
                         Common Eligibility Interface
The internal platform should not care how the organization stores membership.
Every adapter should produce a normalized result such as:
interface EligibilityResult {
  eligible: boolean;
  organizationId: string;
  provider: string;
}
Conceptually:
verifyEligibility(user, election)
The adapter handles the organization-specific implementation.
7. DAO Integration
The DAO integration must support different membership models.
Examples:
Token based
Wallet
 ↓
Token Contract
 ↓
balanceOf(wallet)
 ↓
balance >= required amount
NFT based
Wallet
 ↓
NFT Contract
 ↓
ownsMembershipNFT
Governance contract
Wallet
 ↓
DAO Governance Contract
 ↓
isMember(wallet)
DAO API
Wallet
 ↓
DAO API
 ↓
membership = true
The DAO administrator configures which membership source should be used.
The platform does not independently decide DAO membership.
8. College Integration
For the hackathon, initially support:
Institutional Email
        OR
Student ID
Possible flow:
Student
 ↓
Email / Student ID
 ↓
College verification
 ↓
Eligibility = true
 ↓
Credential issued
For the demo, the college verification backend may be mocked.
Production would use a real college-controlled API/database or identity provider.
9. Layer 3 — Eligibility Verification
This layer answers:
"Is this user eligible to participate in this election?"

It receives the result of the organization's configured eligibility provider.
Example:
DAO Adapter
     ↓
Wallet is DAO member
     ↓
Eligible = true
or:
College Adapter
     ↓
Student is enrolled
     ↓
Eligible = true
This layer establishes the user's eligibility before credential issuance.
10. Layer 4 — DID / Credential
After eligibility is verified, issue a verifiable digital credential.
Think of it as a cryptographically signed digital membership card.
Example concept:
Credential

Organization: ABC DAO
Membership: TRUE
Credential Type: DAO_MEMBER
Validity: 2026
Issuer: Authorized DAO/Platform
The credential should be held by the user.
It should not expose unnecessary personal information to the blockchain.
Possible architecture:
Organization
      ↓
Eligibility verified
      ↓
Credential Issuer
      ↓
Verifiable Credential
      ↓
User wallet/storage
The credential should later be usable to generate a ZK proof.
Do not put:
- Aadhaar
- student ID
- email
- real name
- home address
- private credential data
on-chain.
11. Layer 5 — ZK Proof
ZK = Zero Knowledge.
The purpose of this layer is:
Allow the user to prove eligibility without revealing the underlying identity information.

Conceptually:
PRIVATE INPUTS
 ├── Credential
 ├── Secret
 └── Other private information

PUBLIC INPUTS
 ├── Organization ID
 ├── Election ID
 └── Eligibility commitment/root

          ↓

      ZK CIRCUIT

          ↓

       ZK PROOF
The verifier should be able to determine:
"This user has a valid credential and is eligible."
without learning:
"This is Alice, student ID 12345."
Do NOT implement cryptographic primitives from scratch.
Use established ZK libraries/protocols.
12. Layer 6 — Nullifier / Double Vote Prevention
The ZK proof establishes eligibility.
The nullifier prevents the same credential/identity from voting twice in the same election.
Conceptually:
identity_secret + election_id
             ↓
          Hash
             ↓
        Nullifier
Example:
Alice + Election 1 → Nullifier A
Alice + Election 2 → Nullifier B
If Alice tries voting twice in Election 1:
Election 1 → Nullifier A
                    ↓
             Already used
                    ↓
                  REJECT
The smart contract can maintain a set of used nullifiers.
Important:
Nullifier ≠ complete Sybil resistance.
It prevents repeated use of the same identity/credential.
It does not automatically prevent one human from creating multiple legitimate identities.
For the hackathon, clearly define whether the voting model is:
1 verified organization member = 1 vote
rather than claiming:
1 human = 1 vote
unless an actual Sybil-resistance system is implemented.
13. Layer 7 — Private Voting
The system must prevent the public from linking a voter to their selected candidate.
Avoid:
Wallet A → Candidate A
Instead:
Voter
 ↓
Encrypted/private vote
 ↓
Privacy voting protocol
 ↓
Blockchain
Do not build the advanced cryptographic voting engine from scratch.
Evaluate existing systems such as:
- MACI
- Semaphore
- Vocdoni
Preferred investigation order
Start by evaluating MACI for the private voting/tallying layer.
Semaphore can be useful for anonymous membership/proof functionality.
Vocdoni should also be evaluated as a possible ready-made voting infrastructure.
The final choice must be based on actual compatibility with the rest of the system.
14. Layer 8 — MST Blockchain Layer
The hackathon is sponsored by MST Blockchain.
Use MST's available tooling for blockchain interaction.
Provided resources include:
- MST TypeScript SDK
- MST Python SDK
- MST Vibe Kit
- MST MCP
Use the TypeScript SDK as the primary integration candidate because the main application stack is TypeScript.
Use Vibe Kit for rapid hackathon development where appropriate.
Use MST MCP where MCP/agent integration makes sense.
Do not assume unsupported functionality. Check the current MST documentation/packages before implementation.
15. Smart Contract Layer
Smart contracts enforce important election rules.
Conceptual responsibilities:
Election ID
Organization ID
Voting rules
Start/end time
Nullifiers
Vote commitments/references
Tally verification
Final result
The contract should reject:
Election not started
Election ended
Invalid ZK proof
Already-used nullifier
Invalid vote
Unauthorized action
Election configuration should become immutable/frozen once voting begins, where practical.
16. Tallying / Finalization
The voting protocol should handle the cryptographic tallying where possible.
Flow:
Voting Open
    ↓
Votes submitted
    ↓
Voting Ends
    ↓
Election configuration frozen
    ↓
Valid votes collected
    ↓
Tally
    ↓
Generate ZK tally proof
    ↓
Smart contract verifies proof
    ↓
Election finalized
    ↓
Result published
Example result:
Candidate A: 523
Candidate B: 411
Candidate C: 189
The public should be able to verify the result without seeing every private vote.
For the hackathon, reuse the selected protocol's tally infrastructure instead of building a new ZK tally system.
17. API / SDK / MCP Layer
The platform should expose core functionality through an API.
Conceptual API:
POST /organizations
POST /organizations/:id/elections
POST /eligibility/verify
POST /credentials/issue
POST /proofs/generate
POST /votes
POST /elections/:id/finalize
GET  /elections/:id/results
GET  /elections/:id/audit
The exact API design can change during implementation.
SDK should wrap the API and simplify integration.
MCP can expose operations to AI agents.
Example MCP tools:
create_organization()
configure_eligibility()
create_election()
verify_member()
issue_credential()
generate_vote_proof()
cast_vote()
finalize_election()
get_results()
verify_election()
18. Backend
Use:
Express + TypeScript
Responsibilities:
Organization Service
Election Service
Eligibility Service
Credential Service
Voting Service
Blockchain Service
Audit Service
Conceptually:
Next.js
    ↓
Express API
    ↓
Services
    ├── Organization
    ├── Election
    ├── Eligibility
    ├── Credential
    ├── Voting
    ├── Blockchain
    └── Audit
The backend should orchestrate the system.
It should NOT become the sole source of truth for critical voting state.
19. Database
Use:
PostgreSQL
Do not use generic "SQL" as the database choice.
Use Prisma as the ORM.
Possible tables/entities:
Organization
Election
ElectionRule
EligibilityProvider
CredentialMetadata
UserSession
VoteReference
NullifierStatus
BlockchainTransaction
AuditEvent
Do not store:
Private keys
Plaintext private votes
Unnecessary PII
Raw Aadhaar information
20. Frontend
Use:
Next.js + TypeScript
Two main interfaces:
Voter application
Connect/login
 ↓
Verify eligibility
 ↓
Credential
 ↓
ZK proof
 ↓
Vote
 ↓
Vote accepted
Admin dashboard
Login
 ↓
Organization
 ↓
Create election
 ↓
Configure eligibility
 ↓
Configure voting rules
 ↓
Start election
 ↓
Monitor
 ↓
Finalize
 ↓
View result
 ↓
Audit
21. Authentication & Admin Security
Use:
Authentication
      ↓
JWT/session
      ↓
RBAC
      ↓
Permission check
Possible roles:
Organization Owner
Election Admin
Election Operator
Auditor
Viewer
Sensitive blockchain actions can later use multisig.
Important:
Admins should NOT have routine access to individual private votes.
22. Auditability
The system should provide a public verification page.
Display:
Election ID
Organization
Rules
Start/end time
Contract address
Number of votes
Final tally
Proof verification status
Relevant cryptographic commitments
The user should be able to verify:
"My vote was accepted."
The public should be able to verify:
"The election was finalized correctly."
"The tally proof is valid."
"Duplicate nullifiers were rejected."
"Election rules were not changed after voting started."
23. Identity Recovery
Identity recovery should NOT be part of the normal voting flow.
Normal model:
Organization
    ↓
Knows member identity

Voting system
    ↓
Knows anonymous credential/nullifier

Blockchain
    ↓
Knows cryptographic state
If identity recovery is absolutely required:
Recovery request
      ↓
Strong authorization
      ↓
Threshold/multisig approval
      ↓
Controlled identity disclosure
      ↓
Audit log
Do not implement an unrestricted admin decoder.
For the hackathon, identity recovery can be omitted.
24. Threat Model
Double voting
Defense:
ZK proof
+
Election-specific nullifier
+
Smart contract nullifier registry
Fake DAO membership
Defense:
Verify against configured DAO source
Credential theft
Defense:
- user-controlled storage
- expiration
- revocation/reissuance
- wallet binding where appropriate
Admin abuse
Defense:
- RBAC
- immutable/frozen election configuration
- audit logs
- multisig for sensitive actions
Vote privacy failure
Defense:
- encrypted/private votes
- ZK proofs
- separation of identity and vote records
- established privacy voting protocol
Sybil attack
Must be treated separately from double voting.
Do not claim one-human-one-vote unless actual human uniqueness/Sybil resistance is implemented.
25. Technology Stack
Frontend
Next.js
TypeScript
React
Backend
Node.js
Express
TypeScript
Database
PostgreSQL
Prisma
Blockchain
MST Blockchain
MST TypeScript SDK
MST Vibe Kit
MST MCP
Identity
DID-compatible architecture
Verifiable Credentials
Use established libraries/standards rather than inventing a credential protocol.
ZK
Use an established ZK framework/library/protocol.
Do NOT implement cryptography from scratch.
Private voting
First evaluate:
MACI
Semaphore
Vocdoni
Do not lock the implementation to one protocol until compatibility is confirmed.
Authentication
JWT/session authentication
RBAC
Infrastructure
Use simple hackathon-friendly deployment.
Possible:
Next.js → Vercel
Express → Node-compatible hosting
PostgreSQL → Managed PostgreSQL
The exact hosting provider is not important initially.
26. Recommended Repository Structure
voting-platform/
│
├── apps/
│   ├── web/
│   └── api/
│
├── packages/
│   ├── db/
│   ├── blockchain/
│   ├── identity/
│   ├── zk/
│   ├── voting/
│   ├── eligibility/
│   └── shared/
│
├── contracts/
│
├── docs/
│
└── README.md
Responsibilities
apps/web
    Next.js frontend

apps/api
    Express backend

packages/db
    Prisma/PostgreSQL

packages/blockchain
    MST SDK wrapper

packages/identity
    DID/credential functionality

packages/zk
    ZK proof generation/verification

packages/voting
    Private voting protocol adapter

packages/eligibility
    College/DAO adapters

packages/shared
    Shared TypeScript types

contracts
    Smart contracts
27. End-to-End College Flow
Student
 ↓
College email / Student ID
 ↓
College verification
 ↓
Eligible
 ↓
Credential issued
 ↓
Credential stored by user
 ↓
ZK proof generated
 ↓
Election-specific nullifier
 ↓
Private vote
 ↓
MST Blockchain
 ↓
Smart contract
 ↓
Tally
 ↓
Result
28. End-to-End DAO Flow
DAO User
 ↓
Wallet
 ↓
DAO membership source
 ↓
DAO adapter
 ↓
Eligible
 ↓
Credential
 ↓
ZK proof
 ↓
Election-specific nullifier
 ↓
Private vote
 ↓
MST Blockchain
 ↓
Smart contract
 ↓
Tally
 ↓
Result
29. Election Creation Flow
Admin
 ↓
Authenticate
 ↓
Create organization/election
 ↓
Select eligibility provider
 ↓
Configure voting rules
 ↓
Add candidates/options
 ↓
Set start/end time
 ↓
Deploy/configure contract
 ↓
Publish election
 ↓
Freeze configuration when voting begins
30. Vote Flow
User
 ↓
Credential
 ↓
Generate ZK eligibility proof
 ↓
Generate election-specific nullifier
 ↓
Verify proof
 ↓
Check nullifier
 ↓
Encrypt/private vote
 ↓
Submit to voting protocol
 ↓
Record cryptographic state on blockchain
 ↓
Return vote acceptance/reference
31. Tally Flow
Election closes
 ↓
Freeze election
 ↓
Collect valid votes
 ↓
Tally
 ↓
Generate tally proof
 ↓
Verify proof
 ↓
Finalize smart contract
 ↓
Publish result
 ↓
Enable public audit
32. Build Order
DO NOT build everything at once.
Phase 1 — Basic infrastructure
Build:
Next.js
+
Express
+
PostgreSQL
+
Prisma
Verify:
Frontend → API → Database
Phase 2 — MST integration
Use MST TypeScript SDK/Vibe Kit.
Build:
Backend
 ↓
MST SDK
 ↓
Blockchain
Create a basic election/transaction flow.
Phase 3 — Election management
Build:
Organization
 ↓
Election
 ↓
Candidates
 ↓
Rules
 ↓
Start/end
Phase 4 — Eligibility adapters
Build:
College Adapter
DAO Adapter
Initially support:
College → email/student ID
DAO → wallet membership
Phase 5 — Credential
Build:
Verified user
 ↓
Credential issuer
 ↓
Credential
 ↓
User storage
Phase 6 — ZK
Build/integrate:
Credential
 ↓
ZK proof
 ↓
ZK verifier
The first successful test should be:
Valid credential → proof verifies
Invalid credential → proof fails
Phase 7 — Nullifier
Implement:
identity secret
+
election ID
↓
nullifier
Test:
First vote → accepted
Second vote → rejected
Different election → accepted
Phase 8 — Private voting protocol
Integrate the selected protocol.
Target:
ZK eligibility
+
nullifier
+
private vote
↓
Voting protocol
Phase 9 — Tally
Implement/integrate:
Voting closes
 ↓
Tally
 ↓
Proof
 ↓
Verification
 ↓
Result
Phase 10 — API/SDK/MCP
Expose:
Organization management
Election management
Eligibility
Credentials
Voting
Results
Audit
Phase 11 — Admin dashboard
Build the complete admin workflow.
Phase 12 — Audit + security
Add:
- RBAC
- audit logs
- election freeze
- blockchain transaction display
- proof verification
- error handling
- rate limiting
- secret management
33. What NOT to Build From Scratch
Do NOT independently implement:
Cryptographic primitives
ZK proving system
Advanced encrypted voting protocol
ZK tally cryptography
Blockchain infrastructure
Reuse established systems.
Your differentiation is the integration and voting infrastructure layer that connects:
Organizations
+
Eligibility
+
Credentials
+
ZK
+
Private voting
+
MST Blockchain
+
API/SDK/MCP
+
Auditability
34. Hackathon Scope
The hackathon version should prioritize one complete working path over supporting every possible feature.
Recommended demo:
DAO
DAO admin
 ↓
Configure DAO membership
 ↓
Create election
 ↓
User connects wallet
 ↓
Membership verified
 ↓
Credential
 ↓
ZK proof
 ↓
Nullifier
 ↓
Private vote
 ↓
MST blockchain
 ↓
Tally
 ↓
Result
College
Show the same architecture with:
College email/student ID
instead of wallet membership.
35. Definition of Done
The core MVP is complete when:
- An organization can be created.
- A DAO or college eligibility method can be configured.
- An eligible user can obtain a credential.
- The user can generate a valid ZK eligibility proof.
- An ineligible user cannot generate a valid voting proof.
- A voter receives an election-specific nullifier.
- A second vote using the same identity for the same election is rejected.
- A vote can be submitted privately.
- The transaction/state is recorded on MST Blockchain.
- The election can be finalized.
- The tally can be verified.
- The result can be displayed.
- An outsider can inspect the public audit information.
- The voter's identity is not publicly linked to their vote.
36. Most Important Architectural Rule
Keep these three things separate:
IDENTITY
"Who is this person?"

        ↓

ELIGIBILITY
"Are they allowed to vote?"

        ↓

VOTING
"What did they vote for?"
The organization handles identity/membership.
The platform handles eligibility and privacy-preserving credentials.
The ZK system proves eligibility.
The nullifier prevents repeat voting.
The private voting protocol protects the vote.
MST Blockchain provides the blockchain infrastructure and verifiable on-chain state.
The tally system produces the final verifiable result.
37. Agent Instructions
When implementing this project:
1. Do not rename the project unless explicitly instructed.
2. Do not redesign the architecture without identifying a concrete technical blocker.
3. Prefer existing audited cryptographic/voting infrastructure.
4. Do not invent undocumented MST SDK functionality.
5. Inspect current MST SDK/Vibe Kit/MCP documentation before implementing MST-specific code.
6. Keep secrets in environment variables and never commit credentials.
7. Never expose OAuth client secrets in frontend code or Git.
8. Keep PII off-chain.
9. Do not create an unrestricted admin identity decoder.
10. Clearly separate identity records from anonymous voting records.
11. Use PostgreSQL + Prisma.
12. Use TypeScript across frontend/backend unless a specific library requires another language.
13. Build one complete end-to-end voting flow before adding advanced features.
14. For protocol integrations, wrap the external protocol behind an internal adapter so it can be replaced later.
15. Clearly mark mock/hackathon components so they can later be replaced with production implementations.
The immediate implementation goal is:
Next.js
    ↓
Express
    ↓
PostgreSQL
    ↓
MST Blockchain
    ↓
Basic Election
    ↓
Eligibility
    ↓
Credential
    ↓
ZK Proof
    ↓
Nullifier
    ↓
Private Vote
    ↓
Tally
    ↓
Audit
Build and test each stage before moving to the next.
'''
path = "/mnt/data/project_master_brief_privacy_voting.md"
with open(path, "w", encoding="utf-8") as f:
    f.write(md)
print(path)
print(os.path.getsize(path))