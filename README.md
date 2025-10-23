# BoysInSkirts.org — Project Information

## 1. Executive summary

BoysInSkirts.org (BIS) is an informational and advocacy website that promotes gender equality in clothing and normalizes skirts as an option for boys and men. The public site will combine editorial content with an engaging hero experience: a 3D virtual try-on (VT) system that demonstrates how skirts look on a variety of human models. The project is built around a small, tailored headless CMS that feeds structured data to both the public site and the admin panel. The CMS will also manage human model assets and skirt patterns and expose APIs for the 3D renderer.

Primary goals:
- Deliver a lightweight, performant public website with accessible content and an immersive VT hero.
- Build a flexible headless CMS that allows non-technical editors to author content, manage models, and manage skirt assets.
- Provide an admin panel to manage site content and model/skirt assets.
- Ensure privacy, accessibility, and good performance across devices.

Target launch: staged rollout — MVP with core editorial content and a basic VT model, followed by iterative improvements (skirt generation, customization, analytic events).

---

## 2. Objectives & success metrics

Primary objectives:
- Publish core editorial content (landing, about, blog/articles, FAQs, legal pages).
- Deliver a functioning VT hero that renders 3–6 base human models with selectable skirts and basic fit/pose switching.
- Provide a CMS for editors to manage pages, media, human models, skirt assets.
- Achieve accessible experience (WCAG AA), maintain privacy-compliant tracking, and fast page loads (Lighthouse performance score >= 80).

Success metrics:
- Launch with at least 10 content pages and 3 human models + 4 skirt styles.
- Time-to-interactive on landing page < 3s on 4G.
- Editor workflow to create and publish a page in < 10 minutes.
- >90% of core flows pass accessibility audits.
- Active weekly visitors target (TBD by marketing) and conversion (newsletter signups) baseline.

---

## 3. Audience & user personas

- General public / curious visitors — lightweight information, easy-to-consume content.
- Supporters & advocates — deeper content, shareable assets.
- Editors / Campaign managers — non-technical users who manage content and models.
- Technical admins — manage deployments, assets, and integrations.

---

## 4. Scope

In-scope (MVP):
- Headless CMS with user auth, JWT-based API, content pages (WYSIWYG or structured blocks), media library.
- CMS content types: Page, Article, Static (terms/privacy), Hero configuration (which model/skirt to display).
- Human model management: upload, metadata, preview image, simple pose selector.
- Skirt assets: upload 3D files (glTF), CRUD, preview thumbnails; support for both pre-made 3D models and parametrically generated patterns.
- Main website (React + Three.js): landing page with hero VT, content pages, basic responsive UI components.
- Admin panel (React): CMS UI, model/skirt management, content preview, basic analytics dashboard.
- Basic user tracking (anonymous analytics events) with opt-out / cookie consent.
- Authentication for CMS/Admin with JWT and RBAC (Editor, Admin).
- CI/CD and automated tests for critical paths.

Out-of-scope (for MVP):
- Full ecommerce, payment processing.
- Advanced user customization (body scanner via camera, measurement capture).
- Social features (comments, user accounts for public users).
- Ads network integrations (can be planned for Phase 2).
- Heavy ML-based skirt generation (optional later).

---

## 5. High-level architecture

- Headless CMS (Spring Boot, MySQL) — exposes RESTful JSON APIs for content, models, assets.
- Main Website (React + TypeScript) — client-side app using Tailwind CSS and Material UI for components; Three.js for 3D VT.
- Admin Panel (React + TypeScript) — SPA that consumes CMS APIs; shares UI library.
- Storage: MySQL for structured data; S3-compatible object storage for media and 3D assets.
- CDN: Serve public assets and built frontends via CDN (CloudFront, Cloudflare).
- Auth: JWT issued by CMS for admin panel and internal API auth. Admin UI protected by RBAC.
- Analytics: Privacy-first analytics (Plausible or self-hosted Matomo) + optional server-side event tracking.
- Infrastructure: Dockerized services, Kubernetes or managed container service (ECS/Fargate), Terraform for infra as code (optional).

Diagram (conceptual):
Public Browser <-> CDN -> React app (Three.js) <-> CMS REST API (Spring Boot) -> MySQL, S3
Admin Browser <-> CDN -> Admin React app <-> CMS REST API (auth required)

---

## 6. Tech stack (recommended)

- Backend / CMS: Spring Boot (Java 17+), Spring Security (JWT), Spring Data JPA, MySQL / Aurora MySQL.
- Frontend: React + TypeScript, Vite or Next.js (if SSR desired), Tailwind CSS, Material UI (component lib), Three.js / react-three-fiber for 3D rendering.
- Asset storage & CDN: AWS S3 + CloudFront (or equivalent).
- Auth: JWT; optionally integrate with OAuth2 for external logins (admins only).
- CI/CD: GitHub Actions (build/test/containers), automated deployment to staging & prod.
- Monitoring: Sentry for errors, Prometheus/Grafana for infra metrics, Cloud provider monitoring.
- Analytics: Plausible (privacy-focused) or Matomo (self-host).
- Optional: Node microservice for heavy 3D processing/skirt generation offline.

Rationale: chosen stack balances familiarity, scalability, and the mature Java ecosystem for server side control of CMS data shapes and APIs.

---

## 7. Detailed feature breakdown

7.1 Headless CMS
- Authentication & RBAC:
  - Login (username/password), JWT token issuance, token refresh.
  - Roles: Admin, Editor, Viewer.
- Content types:
  - Page: title, slug, metadata (SEO), hero configuration (model, skirt), content blocks (rich text, image, embed).
  - Article: similar to Page with tags, publication date.
  - Static: Terms, Privacy.
- Custom fields:
  - Simplified ACF-like system for: text, textarea, rich text (HTML/Markdown), image, select, boolean, relation (link to other content assets), nested groups.
  - Ability to define page templates and which fields are editable.
- Media library:
  - Upload, automatic image resizing, thumbnails, metadata, reference counting.
- Human models:
  - CRUD, name, body type tags (slim, average, plus-size), skin tone tags, preview thumbnails, default pose.
  - 3D model references (glTF/GLB) or baked preview images.
- Skirt assets:
  - CRUD, metadata (style, pattern parameters), upload glTF/GLB or SVG patterns if procedural.
  - Preview thumbnails; mapping to skirt anchor points on human models.
- Skirt generator (see section 8).
- API:
  - RESTful endpoints for all content with pagination, filtering, caching headers.
  - Public read endpoints without auth (for public site).
  - Admin endpoints behind JWT.
- Audit & versioning:
  - Basic page history and unpublished drafts; optional versioning for models/assets.
- Webhooks:
  - On publish, trigger cache purge/CDN invalidation.

7.2 Main Website (Public)
- Landing page:
  - Hero with 3D VT: loads base model + default skirt.
  - Model + skirt selector (popovers) with thumbnails.
  - Accessible controls for non-3D fallback (image or video).
- Content pages:
  - Rendered from CMS content blocks; support SEO meta tags and OpenGraph.
- Legal pages (terms, privacy).
- Progressive enhancement: provide content primarily; defer heavy 3D assets behind user interaction or lazy-load.
- Accessibility:
  - Keyboard navigable, semantic HTML, ARIA labels for 3D controls, color contrast.
- Performance:
  - Lazy-load 3D assets, compress models, use LODs, serve via CDN.

7.3 Admin Panel
- Login + MFA optional.
- Content editor:
  - WYSIWYG / structured editor for blocks, preview button (desktop + mobile).
- Model & skirt management UI:
  - Upload, preview, tag, and assign to hero presets.
- Publish workflow:
  - Draft, review, publish; preview as public.
- Analytics dashboard:
  - Top pages, VT engagements, anonymous events like "try-on interactions".
- Role management:
  - Invite users, assign roles.

---

## 8. Skirt generation algorithm (strategy & options)

This is a core differentiator. There are three practical approaches; choice depends on budget and timeline.

Option A — Pre-made 3D assets (MVP)
- Maintain a curated library of glTF/GLB skirt models (different styles, sizes).
- For each human model, provide fitted versions or use simple attach points (skirt parented to hip bone).
- Pros: reliable, faster to implement.
- Cons: less customizable.

Option B — Parametric / procedural skirt generator (recommended Phase 2)
- Use a generator that creates skirt geometry from parameters (length, waist circumference, fullness, pleat count, fabric drape).
- Implementation:
  - Server-side Node or Java microservice that outputs optimized glTF for given parameters.
  - Use cloth-simulation-free approach (baked geometry), or precomputed drape LODs.
- Pros: flexible customization, smaller asset library.
- Cons: requires algorithm development and asset pipeline.

Option C — ML-driven or physics-based simulation (advanced)
- Use machine learning (e.g., conditional generative models) or GPU-based cloth simulation to generate realistic drapes for arbitrary body shapes.
- High complexity and compute; better for later phases or research.

MVP recommendation: Option A for launch — curated glTF models with standardized anchor points and a system to tag compatibility with human models. Plan Option B for Phase 2 for user customization & smaller UX friction.

Integration details:
- Each skirt asset includes metadata: anchor offsets, collisions (optional), recommended scale, compatible body types, and a fallback image.
- CMS exposes generator endpoints (if implemented) that accept parameters and return a downloadable glTF and thumbnail.

---

## 9. Data model (high-level)

Entities:
- User {id, username, email, role, hashed_password, created_at}
- Page {id, title, slug, status, template, fields:json, author_id, published_at}
- Article {id, ...}
- Media {id, key, url, mime, width, height, sizes, created_at}
- HumanModel {id, name, body_type, tags, glb_url, preview_image_id, anchor_map:json}
- SkirtAsset {id, name, style, glb_url, preview_image_id, compatible_models:[ids], parameters_schema:json}
- AuditLog {entity, entity_id, action, user_id, timestamp}

API patterns:
- /api/pages?slug=landing — public read
- /api/human-models — public read (but limited metadata)
- /api/admin/human-models — authenticated CRUD
- /api/skirt-generator (POST parameters) — (optional) returns glTF or job id

---

## 10. Security & privacy

Security:
- Use HTTPS everywhere.
- JWT tokens with reasonable expiry + refresh tokens.
- Rate limiting on public APIs.
- Validation and sanitization for uploaded assets (virus scan, mime checks).
- Least-privilege principle for service accounts.

Privacy:
- Minimal data collection — no personal data from public visitors unless they sign up.
- Analytics: use privacy-first analytics, anonymize IPs, and provide cookie consent UI with opt-out.
- For any tracking related to VT usage, use event aggregation and keep data anonymous (no device-level linkage without explicit consent).
- Provide clear privacy policy explaining usage.

Compliance: design with GDPR/CCPA considerations — DSAR handling process if user data collected.

---

## 11. Accessibility

- Target WCAG 2.1 AA across public site.
- Ensure 3D VT controls have accessible alternatives (simple image fallback, keyboard controls).
- Provide descriptive alt text for images and 3D content descriptions.
- Color contrast checks, focus indicators, semantic markup.

---

## 12. Performance & optimization

- Lazy-load 3D assets and defer heavy scripts until after hero content visible or on user interaction.
- Use compressed glTF (DRACO) and LODs.
- Use HTTP/2 or HTTP/3 via CDN.
- Cache returned API content with ETag/Cache-Control and CDN edge caching.
- Monitor Lighthouse and set budget (bundle size targets).

---

## 13. Testing strategy

- Unit tests: backend (Spring Boot), frontend components.
- Integration tests: API contract tests, end-to-end tests (Playwright or Cypress) for public flows and admin publish workflow.
- Visual regression tests for key pages and VT hero fallback.
- Accessibility testing using axe-core integrated into CI.
- Performance regression checks (bundle sizes, Lighthouse snapshots).

---

## 14. Deployment & infra

- Environments: local dev, staging, production.
- Containerize backend and optional asset-processing services.
- Recommended hosting: managed Kubernetes (EKS/GKE/AKS) or ECS Fargate for smaller ops footprint.
- Use RDS / managed MySQL for database.
- S3 or equivalent for assets + CloudFront/Cloudflare CDN.
- CI/CD: GitHub Actions — build, test, push images, deploy to staging then production with controlled rollout.
- Backups: DB daily, assets replicated.

---

## 15. Roadmap & milestones

Phase 0 — Discovery & prototyping (2–4 weeks)
- Finalize requirements, seed content model, create 3 prototype human models and 3 skirt glTFs.
- Proof-of-concept Three.js VT in React (static assets).

Phase 1 — MVP (8–12 weeks)
- Implement CMS basic CRUD, auth, media library.
- Public site with content pages and landing page VT (pre-made skirts).
- Admin panel: content editing, model/skirt CRUD, publish flow.
- CI/CD, staging environment, analytics + privacy banner.
- Accessibility baseline and smoke tests.

Phase 2 — Feature expansion (8–12 weeks)
- Parametric skirt generator (server-side), new model shapes, advanced previews.
- Performance optimizations, SEO improvements.
- Analytics dashboards, enhanced editor features.

Phase 3 — polish & scale (ongoing)
- Social sharing, multi-language support, possible marketing integrations, ad platform if needed, community features.

Total initial: ~3–4 months to MVP depending on team size.

---

## 16. Team & roles (suggested)

- Product Owner / Campaign Lead (1)
- Tech Lead / Backend Engineer (1)
- Frontend Engineer (1–2) — React + Three.js experience advantageous
- 3D Artist / Technical Artist (1) — produce glTF models, thumbnails, anchors
- QA / Accessibility Tester (0.5–1)
- DevOps (0.5) — infra & deployment
- Optional: ML/Graphics Engineer (later phases) for skirt generator or simulation

---

## 17. Risks & mitigations

- Risk: 3D complexity causes performance or compatibility issues on low-end devices.
  - Mitigation: lazy-load 3D, provide image fallback, compress models, keep hero interaction optional.
- Risk: Skirt fit issues when attaching generic skirts to multiple body shapes.
  - Mitigation: define compatibility matrix; plan procedural generator for Phase 2.
- Risk: Admin/editor UX complexity.
  - Mitigation: user-centered design sessions and simple templates for editors.
- Risk: Privacy/consent mistakes with tracking VT usage.
  - Mitigation: use privacy-first analytics, opt-in for any non-anonymous telemetry.

---

## 18. Acceptance criteria (MVP)

- CMS can create/edit/publish pages and media, and editors can manage human models and skirt assets.
- Landing page displays hero 3D VT with at least 3 models and 4 skirts; controls allow swapping model/skirt.
- Public APIs deliver content within 200ms median response (staging).
- Admin user can log in, create a draft, preview, and publish a page.
- Accessibility: WCAG AA checks for published pages and controls.
- Privacy: cookie banner present; analytics respect opt-out.

---

## 19. Rough cost & effort estimate (high level)

(Assuming mid-senior contractors; cost varies widely by region)

- Discovery & prototyping: 2–4 weeks, 1–2 people.
- MVP build: ~8–12 weeks:
  - Backend: 2–3 engineer-weeks
  - Frontend: 3–5 engineer-weeks
  - 3D assets & integration: 2–3 weeks
  - QA/Accessibility & DevOps: concurrent
- Ongoing monthly maintenance & improvements: 1–2 FTE-equivalents.

We can produce a more detailed sprint plan and cost estimate once team composition and hourly rates are known.

---

## 20. Next steps / recommended immediate actions

1. Approve MVP scope and allocate initial budget & roles.
2. Create a prioritized backlog of pages, models, and skirt styles for the launch.
3. Produce 3 base human models and 4 skirt assets as launch assets.
4. Implement a minimal CMS schema and a proof-of-concept Three.js hero to validate performance and UX.
5. Plan Phase 2 for procedural skirt generation after validating anchor/compatibility model.

---

Appendix A — Example API endpoints (MVP)

- GET /api/pages/:slug
- GET /api/articles?tag=gender-equity
- GET /api/human-models
- GET /api/skirt-assets
- POST /api/admin/login
- GET /api/admin/human-models
- POST /api/admin/skirt-assets
- POST /api/skirt-generator (Phase 2)

Appendix B — Example skirt asset metadata (JSON)
{
  "id": "skirt-001",
  "name": "A-line knee",
  "style": "A-line",
  "glb_url": "https://cdn.example.com/skirts/skirt-001.glb",
  "preview_image": "https://cdn.example.com/skirts/skirt-001.jpg",
  "compatible_body_types": ["slim","average"],
  "anchor": { "bone": "hips", "offset": [0, -0.1, 0], "rotation": [0,0,0] },
  "parameters_schema": null
}

---

If you'd like, I can:
- Turn this into an actionable sprint backlog with epics, user stories, and acceptance criteria.
- Produce a minimal API spec (OpenAPI) for the CMS.
- Draft UI mockups for the hero VT and the admin model/skirt manager.

Which of these would you like next?