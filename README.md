# Securing AI Agents Across Voice and Web

A fourteen-slide [Slidev](https://sli.dev/) talk about patient-facing AI authorization, OAuth standards, and interoperability agreements. Managed with npm.

Greg’s revised speaking notes contain approximately 587 spoken words, excluding reference credits. Per-slide timings are allocated in proportion to note length for a **5:00 runtime with no buffer**, averaging about 117 words per minute. These are estimates, not measured rehearsal times. The original reported run ended at 15:35 and included pauses and editing directions, so that elapsed time is not treated as pure speaking time.

The active talk is in `slides.md`, using the default Slidev theme, Markdown code fences, and Mermaid diagrams with local SVG icons. There is no custom stylesheet or TypeScript setup. Shared utilities enlarge the code and subheadings and give links conventional blue solid underlines. The browser views reuse the Markdown diagram in `slides/browser-diagram.md`, passed a fixed stage using Slidev’s existing Vue support. Each slide has its own note block in `slides.md`; slides five, seven, and nine contain only authentication, delegation, and API-access cues respectively. The shared fragment uses three lines of plain JavaScript to accept its stage; there is no TypeScript setup or duplicate diagram source. Other slides have one short spoken paragraph, except token exchange, which has two.

The conclusion is a single statement: “Existing web standards are a strong foundation for agent security in curated experiences.” It precedes the thank-you slide, which includes a large QR code linking to this repository. The QR is a local PNG in `public/repository-qr.png` and requires no runtime service or extra project dependency. The icon palette is removed. Current icon choices are the chat bubble and server stack.

The closing slide thanks the people writing drafts, sharing explanations, and helping work through real use cases in public. Its speaker notes retain the takeaway for first-party, curated experiences: existing web standards provide the foundation, evolving drafts extend the model, and some implementation agreements remain. Delegation agreements are speaking cues on the token-exchange slide; the prior agreements and shared-conventions closing slides are removed from the active deck.

MCP authorization and service-account-only access each have a separate text slide, allocated 18 and 17 seconds respectively.

The core sequence is browser authentication → access token scope → browser delegation → token exchange → browser API access → resource indicators → voice authentication → First-Party Applications API detail. The scope, resource-indicator, and First-Party Applications detail slides share a left-to-right request → result layout with matching subheadings. API examples, including token exchange, are vertically centered below their titles using the same simple height-and-flex utility pattern as the diagrams. The thank-you credits are vertically centered, with each work linked first and its authors inline afterward. The First-Party Applications detail shows only the challenge request and authorization-code response; code redemption stays in the speaking cues, and the patient verification fields are application-defined.

The per-slide cue times total exactly 5:00 with no buffer. Sources and reference credits are not part of the spoken notes.

The editable Mermaid diagrams use built-in image nodes. A white label-background variable and a few native class/link styles keep icons unboxed and flow arrows thick. The browser and voice diagrams omit policy callouts; those points stay in speaker notes. All Mermaid diagrams use scale 1.25, with compact rank spacing and wrapped flow labels on the authorization diagrams. A flex wrapper centers each diagram. The five local SVGs in `public/icons/` are IBM [Carbon icons](https://github.com/carbon-design-system/carbon/tree/main/packages/icons), licensed under Apache-2.0 (included alongside them). No icon service, custom component, or icon registration is needed.

## Saved version and restore

The styled deck is preserved in `archive/2026-11-17-before-default-styling.zip`, including its stylesheet, icon setup, package files, and speaker notes. `archive/` is ignored by Git. The old long-form draft, unused MCP logo, and earlier experiments also live there.

The plain version before the icon/centering pass is also saved in `archive/2026-11-17-defaults-before-icons.zip`.

To restore the styled deck's presentation files from the project root:

```sh
python3 -m zipfile -e archive/2026-11-17-before-default-styling.zip /tmp/slidev-styled-restore
cp /tmp/slidev-styled-restore/slides.md slides.md
cp /tmp/slidev-styled-restore/style.css style.css
cp -R /tmp/slidev-styled-restore/setup .
```

The earlier blue/green visual direction was an approximation inspired by the [Cologuard website](https://www.cologuard.com/). It is retained in the archive; the active deck uses defaults.

## Technical references and example conventions

- [RFC 6749 — The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749.html): authorization code grant (§4.1) and application-defined scopes (§3.3).
- [RFC 7636 — Proof Key for Code Exchange by OAuth Public Clients](https://www.rfc-editor.org/rfc/rfc7636.html): browser PKCE example.
- [RFC 8176 — Authentication Method Reference Values](https://www.rfc-editor.org/rfc/rfc8176.html#section-2): registers `kba` for knowledge-based authentication. This names the method; it does not standardize the patient-verification fields or imply that these illustrative tokens carry an `amr` claim.
- [RFC 8693 — OAuth 2.0 Token Exchange](https://www.rfc-editor.org/rfc/rfc8693.html): exchange (§2.1), actor (§4.1), scope claim (§4.2), permitted actor (§4.4), and illustrative delegation (Appendix A.2).
- [RFC 8707 — Resource Indicators for OAuth 2.0](https://www.rfc-editor.org/rfc/rfc8707.html): requesting resource targets and audience restriction. The example uses the same URI for resource and audience; the issuer determines the mapping.
- [OAuth 2.0 for First-Party Applications, draft 04](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-first-party-apps-04): challenge request and authorization-code response. This remains an Internet-Draft. The voice diagram groups the challenge and code-redemption steps into the initial flow; RFC 8693 token exchange is a separate request. Phone, date of birth and postal code in the archived API excerpt are the implementation's verification contract, not fields standardized by this draft.
- [Cross-client delegation ADR](https://exas.atlassian.net/wiki/spaces/VAP/pages/5212766293): baseline for the custom initial-request selector and delegation policy. Its parameter is `xes_allowed_delegate_client_id`; the talk uses Greg's broader conceptual model of an eligible actor, which need not be a client. This is a correction to the ADR's client-only terminology, not a claim that the code already implements arbitrary actor nomination.

Request blocks show selected, decoded form parameters, not copy-and-paste HTTP messages. Client authentication, routine parameters, encoding, and redirects are abbreviated. Token blocks show selected decoded claims, with issuer, audience, expiry and signature omitted; they are not complete usable tokens. All identities are fictional and share an issuer for simplicity. Agent client credentials obtain the actor token; the credentials are not placed in the subject claim.

The permitted-actor and actor claims are standard. The initial nomination parameter is custom. Exchange-only authority and onward-delegation restrictions require an agreed policy/profile. Omitting `may_act` does not universally prohibit exchange, and no standard `exchange_only` claim is implied. Coarse scopes are an implementation choice; scopes do not themselves define an object-level policy model.

The token-exchange slide also shows the custom `client_category` claim. In this browser example, the initial token's top-level category is `human_operated_app`, describing the initiating browser client. Its nested `may_act.client_category` is `ai_agent`, describing the intended delegate. The agent token's top-level category and the delegated token's top-level and `act.client_category` values are also `ai_agent`. The category belongs to the relevant client, not to the patient subject; in voice, the initiating client can already be an AI agent. The illustration follows the cross-client ADR model selected for the talk. Claim placements are grounded in `TokenCustomizer` and `ActClaimBuilder`; the excerpts omit other client metadata. The line break before each category value is just JSON whitespace to keep the slide readable.

## Acknowledgments and related reading

The closing slide groups the First-Party Applications authors, Andrea Chiarelli, and contributors to the public MCP authorization discussion. The broader proposal credits are retained here and in the speaker reference notes.

| Work | Authors / contributors |
| --- | --- |
| [OAuth 2.0 for First-Party Applications](https://www.ietf.org/archive/id/draft-ietf-oauth-first-party-apps-04.html) | Aaron Parecki, George Fletcher, Pieter Kasselman |
| [Browserless authentication discussion](https://github.com/modelcontextprotocol/ext-auth/issues/19) | Contributors including calvertj, travishaagen, aaronpk, localden, and WhammyLeaf |
| [On-Behalf-Of Is a Pattern. Token Exchange Is the Standard.](https://auth0.com/blog/on-behalf-of-is-a-pattern-token-exchange-is-the-standard/) | Andrea Chiarelli |
| [Server Profiles proposal](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1004) | Atul Tulshibagwale, Erik Gustavson |
| [On-Behalf-Of User Authorization for AI Agents](https://datatracker.ietf.org/doc/html/draft-oauth-ai-agents-on-behalf-of-user-02) | Thilina Shashimal Senarath, Ayesha Dissanayaka |
| [OAuth Actor Profile for Delegation](https://datatracker.ietf.org/doc/draft-mcguinness-oauth-actor-profile/) | Karl McGuinness |
| [Attenuating Authorization Tokens for Agentic Delegation Chains](https://datatracker.ietf.org/doc/html/draft-niyikiza-oauth-attenuating-agent-tokens-02) | Niki Aimable Niyikiza |
| [WIMSE Architecture](https://datatracker.ietf.org/doc/draft-ietf-wimse-arch/) | Joseph Salowey, Yaroslav Rosomakho, Hannes Tschofenig |

These are credits for public specifications, proposals, writing, and discussion. The linked draft documents carry their own adoption and publication status.

## Running the deck

Requires Node.js 22.22.0 or newer and npm.

Install dependencies and start the local presentation:

```sh
npm install
npm run dev
```

Open [localhost:3030](http://localhost:3030) if your browser does not open automatically.
Edit `slides.md` to write your presentation. Add `---` between slides.

Build the presentation as a static website in `dist/`:

```sh
npm run build
```

To export a PDF locally, install the rendering browser once, then run the export:

```sh
npx --no-install playwright install chromium
npm run export:pdf
```

The PDF is written to `dist/securing-ai-agents-across-voice-and-web.pdf`, without
speaker notes. The export dependency is included in the lockfile. See the
[Slidev export documentation](https://sli.dev/guide/exporting) for other formats and options.

## Automatic PDFs on GitHub

[Publish presentation PDF](.github/workflows/publish-pdf.yml) runs on every branch
push and can also be started manually from the repository's Actions tab.

- Every successful run saves a `presentation-pdf` workflow artifact for 30 days.
- Default-branch pushes also publish a GitHub Release with the PDF attached.
  Each commit gets its own `slides-<commit>` tag, preserving previous versions.
  Rerunning the same commit replaces that release's PDF.
- The release uses GitHub's built-in token with `contents: write`; no custom secret
  or hosting service is needed. Repository visibility controls who can download it.

After the workflow publishes its first release, download the PDF from
[the latest release](https://github.com/gtaylor-exactsciences/QCon-Lightning-Talk-Securing-AI-Agents-Broser-and-Voice/releases/latest),
or use its stable direct-download URL:

```text
https://github.com/gtaylor-exactsciences/QCon-Lightning-Talk-Securing-AI-Agents-Broser-and-Voice/releases/latest/download/securing-ai-agents-across-voice-and-web.pdf
```

The workflow installs Chromium on its Ubuntu runner and exports one page per
slide. Generated PDFs stay in ignored `dist/`; archives remain ignored as well.

## Rehearsal source checks

The notes use the cross-client ADR as the delegation baseline and Greg’s correction that the eligible delegate need not be a client. The comparison is about implementation agreements; no source is assumed to supersede another based on its date or checkout state.

- [Cross-client delegation ADR](https://exas.atlassian.net/wiki/spaces/VAP/pages/5212766293): initial delegate nomination, exchange-only authority, and controls on onward delegation.
- [Canonical subject and client-category ADR](https://exas.atlassian.net/wiki/spaces/VAP/pages/4484104193): classification of AI agents, backend services, and human-operated applications. The talk uses abbreviated identity examples, not a complete token contract.
- [Browser integration design](https://exas.atlassian.net/wiki/spaces/VAP/pages/5117903236): authorization code flow and PKCE with replaceable patient authentication.
- [Voice architecture](https://exas.atlassian.net/wiki/spaces/VAP/pages/4129095681): browserless verification followed by code redemption and separate delegation. For request/response spelling, the talk follows draft 04 and `AuthorizationChallengeEndpointFilter`, as agreed, rather than the diagram’s alternate continuation shapes.
- The implementation checks used `ResourceIndicatorPolicy`, `DelegationClientSettings`, `TokenCustomizer`, and `ActClaimBuilder` in the reference repository. Resource policy and explicit subject/actor identities matter even when the agent has its own guardrails; these controls do not establish that all AI behavior is safe.
- [RFC 8707 §2](https://www.rfc-editor.org/rfc/rfc8707.html#section-2) identifies protected resources using absolute URIs; a URL is one choice, not a requirement. A resource URI is not a general-purpose selector for a patient or delegate.
- The scopes in this example are coarse and application-defined. OAuth scopes do not by themselves standardize fine-grained permission semantics across systems.

The conference reference mentioned during the run was [How Airbnb Guardrailed Its AI Customer Support Agent](https://qconsf.com/presentation/nov2026/how-airbnb-guardrailed-its-ai-customer-support-agent), by Weiping Peng, scheduled Monday, November 16, at 11:45. It remains optional reading rather than an added aside in the five-minute notes.

The rehearsal capture, reported timing checkpoints, shortened notes, and pre-edit snapshot are preserved in the ignored `archive/` directory.
