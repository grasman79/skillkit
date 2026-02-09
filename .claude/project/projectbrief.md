# [Project Name]

**Version:** 1.0.0
**Last Updated:** [Date]
**Status:** Planning

---

## Overview

### What is this project?
[Describe what your project does in 2-3 sentences]

### Who is it for?
[Describe your target users/audience]

### Core Value Proposition
[What problem does this solve? Why should users care?]

---

## Tech Stack

### Frontend
- **Framework:** [Next.js / TanStack Start / Remix / SvelteKit / Other]
- **UI Library:** [Shadcn UI / Kibo UI / Motia / Other]
- **Styling:** [Tailwind CSS / Other]
- **State Management:** [React Query / Zustand / Context / Other]
- **Icons:** [HugeIcons / Lucide / Heroicons / Other]

### Backend / API
- **Approach:** [Next.js Server Actions / Convex Functions / tRPC / Express / Other]
- **Validation:** [Zod / Yup / Joi / Other]
- **Error Handling:** [How you handle errors]

### Database
- **System:** [Supabase / Convex / Firebase / Prisma + PostgreSQL / Other]
- **ORM/Client:** [Prisma / Drizzle / Convex Client / Supabase Client / Other]
- **Real-time:** [Yes/No - specify if using real-time features]

### Authentication
- **Provider:** [Better Auth / Supabase Auth / Clerk / NextAuth / Custom]
- **Methods:** [Email/Password, OAuth (Google, GitHub, etc.), Magic Links, OTP]
- **Session Management:** [JWT / Cookies / Other]

### AI Integration (if applicable)
- **SDK:** [Vercel AI SDK / OpenAI / Anthropic / Other]
- **Use Cases:** [What AI features will you build?]
- **Models:** [GPT-4, Claude, Llama, etc.]

### Email (if applicable)
- **Service:** [Resend / SendGrid / Other]
- **Use Cases:** [Transactional emails, notifications, marketing, etc.]

### Deployment
- **Platform:** [Vercel / Netlify / Cloudflare / AWS / Other]
- **CI/CD:** [GitHub Actions / Vercel / Netlify / Other]
- **Environment:** [Serverless / Edge / Container / Traditional Server]

### Development Tools
- **Package Manager:** [pnpm / npm / yarn / bun]
- **TypeScript:** [Yes/No - strict mode?]
- **Linting:** [Biome / ESLint / Other]
- **Testing:** [Jest / Vitest / Playwright / Other]
- **Code Quality:** [Ultracite / Biome / ESLint + Prettier / Other]

---

## Core Features

### Feature 1: [Feature Name]
**Priority:** [High / Medium / Low]
**Description:** [What does this feature do?]
**User Story:** As a [user type], I want to [action] so that [benefit]

**Key Components:**
- [ ] [Sub-feature or component 1]
- [ ] [Sub-feature or component 2]
- [ ] [Sub-feature or component 3]

---

### Feature 2: [Feature Name]
**Priority:** [High / Medium / Low]
**Description:** [What does this feature do?]
**User Story:** As a [user type], I want to [action] so that [benefit]

**Key Components:**
- [ ] [Sub-feature or component 1]
- [ ] [Sub-feature or component 2]
- [ ] [Sub-feature or component 3]

---

### Feature 3: [Feature Name]
**Priority:** [High / Medium / Low]
**Description:** [What does this feature do?]
**User Story:** As a [user type], I want to [action] so that [benefit]

**Key Components:**
- [ ] [Sub-feature or component 1]
- [ ] [Sub-feature or component 2]
- [ ] [Sub-feature or component 3]

---

[Add more features as needed]

---

## User Roles & Permissions

### Role 1: [Role Name] (e.g., Admin)
**Permissions:**
- ✅ [Permission 1]
- ✅ [Permission 2]
- ✅ [Permission 3]

**Use Cases:**
- [What can this role do?]

---

### Role 2: [Role Name] (e.g., Standard User)
**Permissions:**
- ✅ [Permission 1]
- ✅ [Permission 2]
- ❌ [What they cannot do]

**Use Cases:**
- [What can this role do?]

---

[Add more roles as needed]

---

## Database Schema

### Main Entities

#### Entity 1: [Table/Collection Name]
```
Fields:
- id: UUID/String (primary key)
- [field_name]: [type] - [description]
- [field_name]: [type] - [description]
- created_at: timestamp
- updated_at: timestamp
```

**Relationships:**
- [Relationship to other entities]

**Security:**
- [RLS policies / Access control rules]

---

#### Entity 2: [Table/Collection Name]
```
Fields:
- id: UUID/String (primary key)
- [field_name]: [type] - [description]
- [field_name]: [type] - [description]
- created_at: timestamp
- updated_at: timestamp
```

**Relationships:**
- [Relationship to other entities]

**Security:**
- [RLS policies / Access control rules]

---

[Add more entities as needed]

---

## API Endpoints / Functions

### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `GET /api/auth/me` - Get current user

### [Feature Group 1]
- `GET /api/[resource]` - List all [resources]
- `GET /api/[resource]/:id` - Get single [resource]
- `POST /api/[resource]` - Create [resource]
- `PUT /api/[resource]/:id` - Update [resource]
- `DELETE /api/[resource]/:id` - Delete [resource]

### [Feature Group 2]
- [List your API endpoints or function names]

---

## Environment Variables

### Required
```env
# Database
DATABASE_URL=[Your database connection string]

# Authentication
AUTH_SECRET=[Random secret for session encryption]
AUTH_URL=[Your app URL]

# [Service Name]
[SERVICE]_API_KEY=[API key]

[Add all required env vars]
```

### Optional
```env
# [Optional feature]
[OPTIONAL_VAR]=[Value]

[Add all optional env vars]
```

---

## Special Requirements

### Performance
- [ ] Target page load time: [X seconds]
- [ ] Support concurrent users: [X users]
- [ ] Database query optimization required: [Yes/No]
- [ ] Caching strategy: [Describe if applicable]

### Security
- [ ] GDPR compliance required: [Yes/No]
- [ ] Data encryption: [At rest / In transit / Both]
- [ ] Rate limiting: [Specify limits]
- [ ] Input validation: [All inputs / Specific inputs]

### Accessibility
- [ ] WCAG compliance level: [A / AA / AAA]
- [ ] Screen reader support: [Yes/No]
- [ ] Keyboard navigation: [Yes/No]
- [ ] Color contrast requirements: [Specific requirements]

### SEO (if applicable)
- [ ] Meta tags optimization: [Yes/No]
- [ ] Sitemap generation: [Yes/No]
- [ ] Open Graph tags: [Yes/No]
- [ ] Structured data: [Yes/No]

### Internationalization (if applicable)
- [ ] Multi-language support: [Yes/No]
- [ ] Supported languages: [List languages]
- [ ] RTL support: [Yes/No]
- [ ] Locale-specific formatting: [Dates, numbers, currency]

---

## Design System (if applicable)

### Color Palette
- Primary: [Color + hex code]
- Secondary: [Color + hex code]
- Accent: [Color + hex code]
- Error: [Color + hex code]
- Success: [Color + hex code]

### Typography
- Font Family: [Font name]
- Headings: [Font + sizes]
- Body: [Font + sizes]
- Code: [Monospace font]

### Spacing Scale
[Define if you have specific spacing requirements]

### Component Patterns
[Document any specific UI patterns you want to enforce]

---

## Third-Party Integrations

### Integration 1: [Service Name]
**Purpose:** [Why are you integrating this?]
**Use Cases:** [What will you use it for?]
**API Documentation:** [Link to docs]

### Integration 2: [Service Name]
**Purpose:** [Why are you integrating this?]
**Use Cases:** [What will you use it for?]
**API Documentation:** [Link to docs]

[Add more as needed]

---

## Development Phases

### Phase 1: Foundation (Week 1-2)
- [ ] Project setup
- [ ] Authentication
- [ ] Database schema
- [ ] Basic UI components

### Phase 2: Core Features (Week 3-4)
- [ ] [Feature 1]
- [ ] [Feature 2]
- [ ] [Feature 3]

### Phase 3: Advanced Features (Week 5-6)
- [ ] [Advanced feature 1]
- [ ] [Advanced feature 2]

### Phase 4: Polish & Launch (Week 7-8)
- [ ] Testing
- [ ] Performance optimization
- [ ] Documentation
- [ ] Deployment

[Adjust timeline to your needs]

---

## Success Metrics

### Technical Metrics
- Code coverage: [Target %]
- Build time: [Target time]
- Page load time: [Target time]
- Lighthouse score: [Target score]

### Business Metrics
- User registration rate: [Target]
- Feature adoption: [Target]
- User retention: [Target]
- Performance SLAs: [Define if applicable]

---

## Known Constraints

### Technical Constraints
- [Any technical limitations]
- [Browser support requirements]
- [Device support requirements]

### Budget Constraints
- [API usage limits]
- [Hosting budget]
- [Third-party service limits]

### Timeline Constraints
- [Launch deadline]
- [Milestone dates]

---

## Questions & Decisions Needed

- [ ] [Question 1 that needs answering]
- [ ] [Question 2 that needs answering]
- [ ] [Decision 1 that needs to be made]
- [ ] [Decision 2 that needs to be made]

---

## References & Resources

- [Link to designs/mockups]
- [Link to user research]
- [Link to competitor analysis]
- [Link to technical specs]
- [Link to brand guidelines]

---

**Notes:**
[Any additional notes, context, or important information that doesn't fit above]

---

*This document is the source of truth for the project. Update it as requirements evolve.*
