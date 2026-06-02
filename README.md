# samlSSO
This plugin is a full rewrite by Chris Gralike of Derrick Smith's initial PHPSAML plugin for GLPI. This plugin has evolved quite a bit since then and is fully redesigned and rewritten to be compatible with GLPI 11. It now supports multiple SAML Identity Providers (IdPs), implements advanced user right rules, Just-In-Time (JIT) provisioning, and more. 

The plugin is fully configurable from the GLPI UI and doesn't require any coding skills. It uses GLPI core components where possible for maximum compatibility and maintainability. It utilizes Composer for quick 3rd party library updates if security issues require it. It follows PSR best practices and, most importantly, is written with a "security-by-design-by-default" approach to help you visually identify security issues.

## Key Capabilities & Features
* **Multiple SAML Identity Providers**: Configure and manage multiple IdPs simultaneously.
* **Just-In-Time (JIT) Provisioning**: Automatically create and update GLPI users dynamically on successful SAML login.
* **Flexible Claim Mapping**: Bind custom SAML assertion attributes and claims directly to GLPI user fields.
* **User Association & Authorization Rules**: Assign profiles, groups, and entities to users based on SAML claims.
* **Security Hardening**: Replay attack protection using assertion ID database tracking, strict time-window verification, and state-machine-backed login/logout flows.
* **Extensive Logging & Audit Trails**: Thorough logging of phase-by-phase login stages for security validation and SIEM monitoring.
* **Configuration Backup & Restore**: Built-in backup functionality to easily export and restore plugin configuration state.
* **Proxy Support**: Native compatibility with TLS-terminating proxies (Kubernetes Ingress, Traefik, HAProxy) using the "Requests Proxied" configuration setting.
* **Continuous Live Update Improvements**: Ongoing architectural updates ensuring seamless configuration adjustments and session synchronization.
* **Localizations**: Supports automated translation workflows to localize the interface for multiple languages.

## Feedback
I am very interested in your challenges and ideas. Want to contribute those? Look for issues with the label 'Public feedback wanted' or create a feedback issue yourself. I would love to engage with you!

## Current Focus
* Hardening the plugin security state machine
* Translation quality and localization coverage (https://app.transifex.com/quinquies/samlsso/)
* Usability optimization and remote logout handling
* SCIM provisioning capability research
* Enhancing the audit log state table for SIEM security monitoring

## Documentation
* Officially supported by Teclib: https://glpi-plugins.readthedocs.io/en/latest/saml/requirements.html
* Further documentation is available on the Wiki: https://github.com/DonutsNL/samlsso/wiki
* [Contributing & Agentic Maintenance](docs/CONTRIBUTING.md)
* [Architecture Decision Records (ADRs)](docs/ADR/)

> [!NOTE]
> *Admin Interface Images Notice*: The screenshots in the wiki and documentation will be renewed to reflect the latest version once updates to the configuration form are completed.

## Building Awareness About the OSS Funding Gap
- 4000+ downloads and counting.
- Hours spent maintaining and supporting glpisaml/samlsso for the past 3 years: +1000h and counting. Coffees received YTD: 29. Hourly compensation for efforts: €0.145.
- I am not in the begging-for-money business, but I do want to build some awareness about the OSS funding gap problem. Be honest about the benefits and consider supporting the OSS projects you are using and making money off of (like GLPI). Building quality software is time-consuming and expensive!

## Want to support my work?
- Star ⭐ my repo and contribute to my stargazer achievement. 
- Want to do more? I just love coffee: https://www.buymeacoffee.com/donutsnl
- Consider donating to codeberg.org to keep Europe's open source movement going.

## Contribute, or learn to code yourself?
Join our Discord at: https://discord.gg/KyMdkqJcGz
Have coding experience (or are learning to code) and want to add meaningful changes and additions? First start by forking this repository and creating pull requests. Address any feedback you receive to see your pull request merged. If you prove to be consistent, you can request repository access as a contributor to help build a great tool. To share an idea, please create an issue outlining the idea or bug.

**Coding:**
- [Follow PSR where possible](https://www.php-fig.org/psr/)
- Use a decent IDE and consider using plugins like:
- Gitlense (intelephense);
- PSR4 compliant namespace resolver;
- Composer integration;
- Xdebug profiler;
- SonarLint;
- Twig language support;

# Credits
OSS depends on community effort! So honor where honours are due 🫶:
- Raul, @gambware, Koen, Marc-henri, Vijay Nayani, Fabio Grasso, for supporting  the OSS-community.
- @MikeDevresse, @eduardomozart, @Neozlag, @tomas321 for providing fixes to the codebase.
- @andreaPress for figuring out and sharing the docker config needed.
- @SpyK-01 for licensing and sharing the logo via https://elements.envato.com/letter-shield-gradient-colorful-logo-XZ7LYCM.
- @dollierp for adding a cleanup task
- Translations: @CTparental, Alan Lehoux (sp), Achraf Chico (fr), Eduardo Peres (us), Jonathan Ronquillo (sp), Achraf Oueldelferraga (fr), Joaquin Etchegaray (sp), Soporte Infrastructura (sp).
- Number of downloads so far: https://hanadigital.github.io/grev/?user=DonutsNL&repo=samlsso (not counting codeberg downloads +3K)


# My thoughts (ADR) on SAML2 versus oAuth 
While OAuth 2.0 combined with OpenID Connect (OIDC) is widely regarded as the modern standard for authentication, choosing between it and SAML depends heavily on the specific architecture of the application being secured.

Modernity and Microservices OAuth/OIDC was designed for modern, distributed architectures. It uses lightweight JSON Web Tokens (JWTs), which are easy for different programming languages to parse. This makes OIDC ideal for Single Page Applications (SPAs), mobile apps, and especially microservice environments, where a stateless token needs to be passed efficiently between many different services to authorize requests. SAML, by contrast, relies on heavy XML protocols, which are cumbersome and inefficient to process in high-speed microservice meshes.
 
OAuth is a **flexible** framework offering various implementation "flows." This flexibility is powerful but introduces the risk of implementation errors; choosing the wrong flow for a use case can create security vulnerabilities. SAML is a **rigid protocol**. It dictates a strict, formalized structure for authentication exchange. While initial configuration is notoriously difficult (requiring precise exchange of XML metadata and certificates), this rigidity means that once configured, the process is robust and offers few opportunities to deviate from a secure path.

**Security Considerations:** 
To mitigate token theft and Replay attacks, both protocols rely on digital signatures to validate authenticity. However, a validly signed token or assertion, if stolen, can potentially be replayed by an attacker.

OAuth/OIDC historically suffered from token theft risks in client-side browsers via XSS attacks. While modern best practices (like backend-for-frontend patterns) mitigate this, high-security scenarios sometimes require advanced, optional configurations like mutual TLS (mTLS). mTLS binds a token to a specific client certificate (PKI), ensuring that a stolen token cannot be replayed by an attacker without the corresponding private key. SAML inherently processes its heavy XML assertions on the backend, reducing client-side surface area. It also has built-in, mandatory mechanisms against replay attacks, such as strict timestamp windows and unique assertion IDs that the Service Provider tracks.

**Best for GLPI imho:** 
GLPI is fundamentally a classic, server-side monolithic (php) application, not a distributed microservice architecture. It does not (yet) require the lightweight JSON token passing that makes OIDC shine in modern apps. Because GLPI handles its logic and sessions almost entirely on the backend, the server-side processing model of SAML is a natural fit. Furthermore, for an internal IT service management tool where stability and strict security controls are paramount, the rigidity of the SAML protocol is an asset, not a drawback. It provides a mature, "locked-down" authentication mechanism that aligns perfectly with GLPI's monolithic architecture and use cases, without the complexity of navigating modern OAuth security flow or advanced mTLS/PKI configurations.