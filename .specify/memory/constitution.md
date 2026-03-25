<!--
Sync Impact Report:
- Version change: [new] → 1.0.0 (initial version)
- Added sections: All sections (new constitution)
- Chrome Extension best practices incorporated
- Templates requiring updates: ⚠ pending validation
- Follow-up TODOs: None
-->

# N8N Chrome Extension Constitution

## Core Principles

### I. Manifest V3 Security-First (NON-NEGOTIABLE)
All extension development MUST use Manifest V3 architecture. Service workers replace background pages for enhanced security and performance. No remotely hosted code execution allowed - all logic must be bundled within the extension package. This principle ensures compliance with Chrome Web Store requirements and modern security standards.

**Rationale**: Manifest V3 is mandatory for Chrome Web Store acceptance and provides critical security improvements including removal of remote code execution vulnerabilities and enhanced content security policies.

### II. Least Privilege Permissions
Extensions MUST request only the minimum permissions necessary for functionality. Permissions are requested incrementally as features are added, with clear justification documented for each permission. Host permissions are scoped to specific domains when possible rather than using broad patterns.

**Rationale**: Minimizing permissions reduces attack surface, improves user trust, and aligns with Chrome's security model requiring explicit user consent for sensitive operations.

### III. Secure Code Architecture
All JavaScript, WebAssembly, and CSS code MUST be included in the extension bundle. No external code execution via eval(), new Function(), or remote script loading. Content scripts, service workers, and UI components are architecturally separated with clear boundaries and communication protocols.

**Rationale**: Bundled code prevents supply chain attacks and ensures all code undergoes Web Store review. Architectural separation enables better security isolation and debugging.

### IV. Content Security Policy Compliance
Extensions MUST enforce strict Content Security Policy that disallows unsafe-inline, unsafe-eval, and remote resource loading. All dynamic content generation uses safe DOM manipulation methods. CSP violations are treated as security failures requiring immediate resolution.

**Rationale**: Strict CSP prevents XSS attacks and code injection vulnerabilities that could compromise user data or browser security.

### V. Privacy-First Development
Extensions MUST minimize data collection, clearly document all data usage, and implement data retention policies. User data stays on-device when possible. Any data transmission requires explicit user consent with clear explanations of purpose and scope.

**Rationale**: User privacy is paramount for Web Store compliance and user trust. Privacy-first design prevents regulatory violations and builds sustainable user relationships.

## Security Requirements

Extensions undergo security review for each release covering permission usage, data handling, and code architecture. Security testing includes CSP validation, permission auditing, and dependency scanning using tools like npm audit and Snyk. All third-party dependencies are vetted for known vulnerabilities before inclusion.

Vulnerability disclosure process requires immediate response to security reports with patches deployed within 48 hours for critical issues. Security documentation is maintained covering threat models, security controls, and incident response procedures.

## Quality Assurance

Automated testing pipeline includes unit tests for business logic, integration tests for Chrome API usage, and end-to-end tests covering user workflows. Testing covers multiple Chrome versions and extension update scenarios.

Code review process requires security-focused review with specific attention to permission usage, data handling, and CSP compliance. Performance testing ensures service worker efficiency and minimal resource consumption.

Release process includes staging environment testing, security scan validation, and incremental rollout monitoring for production issues.

## Governance

Constitution supersedes all other development practices and guides all technical decisions. All code changes must demonstrate compliance with core principles through automated checks and manual review.

Amendment process requires documented security impact assessment, stakeholder approval, and migration plan for existing code. Emergency amendments allowed for critical security issues with retrospective documentation within 7 days.

Development guidance maintained in extension-specific documentation covering Chrome API usage patterns, security implementation details, and debugging procedures.

**Version**: 1.0.0 | **Ratified**: 2026-03-25 | **Last Amended**: 2026-03-25